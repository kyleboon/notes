## Copy config from one app to another
. Export existing app’s variables to config.txt
heroku config -s -a existing-heroku-app > config.txt
2. Review and push to another app
cat config.txt | tr '\n' ' ' | xargs heroku config:set -a new-heroku-app
