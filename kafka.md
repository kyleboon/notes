cat - starts at the beginning, emits all records and exits when it hits the end of the topic - uses kcat

  kcat -C -e -q -b <broker> -t <topic>-X security.protocol=ssl -X ssl.certificate.location=<<location of pemfile>> 
  
first - emits one record from the beginning (or given -o <offset> and -p <partition>). same as using "-c 1"

  kcat -C -e -q -b <broker> -t <topic>-X security.protocol=ssl -X ssl.certificate.location=<<location of pemfile>>-c 1

tail - starts at the end of the topic and continues listening for new events, unbuffered output - uses kcat

  kcat -C -o end -q -b <<broker>> -t <<topic>> -X security.protocol=ssl -X ssl.certificate.location=<<path_to_pem>> 

    -p <partition> - only emit a specific partition
    -o <offset> - start at a specific offset
    -o < -offset> - negative offsets are relative to the last offset on the partition
    -o "s@$(date -d '36 hours ago' '+%s')000" - start at the offset specific number of millis ago
                                                 (use gdate from "brew install coreutils" on OSX)
    -c <count> - only emit the number of records in count
    -J - JSON formatted output
    -f '<format>' - format, ex: -f '%k\t%o\t%p\n' emits the key offset and partition all tab separated
      - %k - key
      - %o - offset
      - %p - partition
      - %h - event headers
      - %T - message timestamp (millis since epoch UTC)
      - %S - the size of the body in bytes, -1 means a tombstone
      - %s - the actual body

search - search for all events for a specific key on the topic
         you can limit your search with to:
         -p <partition #> - a specific partition
         -o <offset> - searching from a specific offset
         -v - verbose will show the kcat offset/partitions that are found with that key

  franz search <environment> <topic-filter> <key>

kcat -C -e -q -b <broker> -t <topic> -f '%k\t%o\t%p\n' | awk -F'\t' -v find_key="$find_key" '$1 == find_key { print $2, $3}' | while read -r offset partition; do kcat_command -C -e -q $kc_args -o "$offset" -p "$partition" -c 1 done

config - show the topic config properties (formatted version of what is in describe)

  kafka-topics.sh --bootstrap-server <broker> --topic <topic> --describe

describe - shows the leader broker and replicas as well as which replicas are in sync - uses apache kafka-topics

  kafka-topics.sh --bootstrap-server <broker> --topic <topic> --describe

metadata - alternate to describe, show broker/partition details and which are in sync - uses kcat

  kcat -L -b <broker> -t <topic>

size - shows the partition, broker, and size on disk for each leader and replica in the cluster

  kcat -L -b <broker>
 tail -1
 sort -n
 jq -rc '
          .brokers[] |
          .broker as $broker |
          .logDirs[].partitions[] |
          (.partition | capture("-(?<part>\\d+)$") | .part) as $partition |
          [
            $partition,
            $broker,
            (.size/1024/1024 | round | tostring) + "M"
          ] |
          @tsv
        '
 kafka-log-dirs.sh --bootstrap-server <broker> --topic-list <topic> --describe

lag - see the lag for a group ID on a kafka topic - uses kt

  kt group -brokers <broker> -topic <topic> -group <group-id>


#checking last message timestamp
function check_last_message_timestamp() {
    set -eo pipefail

    function echoerr() {
      echo "$@" 1>&2
    }

    hours_ago=$(date +%s -d '4 hours ago');
    if [ "$#" -ne 3 ]; then
        echoerr "Usage: ${0##*/} broker topic partition_count"
        exit 1
    fi

    broker=$1
    topic=$2
    partition_count=$3
    max_partition=$((partition_count-1))
    partitions_with_no_recent_message=()

    for P in $(seq 0 $max_partition); do 
        last_message_ms_ago=$(kafkacat -q -C -b "$broker" -t "$topic" -e -p "$P" -o -1 -f '%T')

        if ((last_message_ms_ago < hours_ago)); then
             partitions_with_no_recent_message+=("$broker $topic $P has not had a message produced since $last_message_ms_ago")
        fi
    done
    
    if ((${#partitions_with_no_recent_message[@]} > 0)); then
        for err in "${partitions_with_no_recent_message[@]}"; do
            echo "$err"
        done
        exit 1
    fi
}

## compaction details
export DATA_DIR=data/check_compaction
mkdir -p "$DATA_DIR"

function info() {
  echo "$(date +"%Y-%m-%dT%H:%M:%S") - $@" >&2
}

function save_topic_event_metadata() {
  local env=$1
  local topic=$2
  local partition=${3:-0}
  
  local file="${DATA_DIR:-data/check_compaction}/${env}_${topic}_${partition}"
  
  info "writing $topic partition $partition to: $file"
  # for every event on that partition, write out the key, offset, partition, event timestamp, and message size (-1 is a tombstone)
  franz cat "$env" "$topic" -p "$partition" -f '%k\t%o\t%p\t%T\t%S\n' > $file
  info "done writing: $file"
  echo "$file"
}

function save_full_topic_event_metadata() {
  local env=$1
  local topic=$2
  
  local file="${DATA_DIR:-data/check_compaction}/${env}_${topic}_full"
  
  info "writing $topic to: $file"
  # for every event on that topic, write out the key, offset, partition, event timestamp, and message size (-1 is a tombstone)
  franz cat "$env" "$topic" -f '%k\t%o\t%p\t%T\t%S\n' > $file
  info "done writing: $file"
  echo "$file"
}

function compaction_report() {
  local in_file=$1
  local sort_file="${in_file}_sorted"
  local bucket_date_format=${2:-"%Y-%m-%d"}
  
  info "sorting $in_file"
  # first field is a string (the key) second is the offset numeric, use version sorting to get the right order
  sort -rV "$in_file" > "$sort_file"
  
  info "parsing $bucket_date_format buckets from $sort_file"

  # use gawk for the asorti function that mawk doesn't have 
  < "$sort_file" gawk -b -v event_bucket_date_format=$bucket_date_format '
    BEGIN { tombstones = 0; total_duplicate_keys = 0; total_duplicate_tombstones = 0; compactable_events = 0; last_key=""; }
    {
      total_events++; 
      
      event_bucket=strftime(event_bucket_date_format, substr($4, 1, 10))

      events_in_bucket[event_bucket]++

      if ($1 != last_key) {
         new_key = 1;
         total_unique_keys++
         new_keys_in_bucket[event_bucket]++
      } else {
         new_key = 0;
         total_duplicate_keys++
         duplicate_keys_in_bucket[event_bucket]++
      }
      
      last_key=$1

      if ($5 == -1) { 
        is_tombstone = 1;
        total_tombstones++;
        tombstones_in_bucket[event_bucket]++

        if (new_key == 0) {
          total_duplicate_tombstones++
          duplicate_tombstones_in_bucket[event_bucket]++
        }
      } else {
        is_tombstone = 0;
      }

      if (is_tombstone == 1 || new_key == 0) {
        total_compactable_events++
        compactable_events_in_bucket[event_bucket]++
      } 
    }

    END {
      printf "total events: %s\nunique keys: %s\nduplicate keys: %s\ntombstones: %s\nduplicate tombstones: %s (tombstone AND duplicate key)\ncompactable events: %s  (tombstone OR duplicate key)\n", 
         total_events, total_unique_keys, total_duplicate_keys, total_tombstones, total_duplicate_tombstones, total_compactable_events;

      printf "%15s %12s %12s %12s %12s %12s %12s %19s\n", "bucket", "events", "compactable", "%_eligible", "new_keys", "duplicate", "tombstone", "duplicate_tombstone"  

      # sort the indicies of this array (the buckets) into order
      bucket_count = asorti(events_in_bucket, sorted_buckets)

      for(i = bucket_count; i > 0; i--) {
        bucket = sorted_buckets[i]
        events = events_in_bucket[bucket]
        compactable_events = compactable_events_in_bucket[bucket]
        percent_eligible = 100.0 * (compactable_events/events)
        new_keys = new_keys_in_bucket[bucket]
        duplicate_keys = duplicate_keys_in_bucket[bucket]
        tombstones = tombstones_in_bucket[bucket]
        duplicate_tombstones = duplicate_tombstones_in_bucket[bucket]

        printf "%15s %12d %12d %11d%% %12d %12d %12d %19d\n", bucket, events, compactable_events, percent_eligible, new_keys, duplicate_keys, tombstones, duplicate_tombstones 
      } 
    }
  '
}

