# Nightscout_Docker-Compose
A comprehensive Docker-Compose for a self-hosted Nightscout CGM (Glucose Monitoring) service
<H6 align="right"><sub>I think I'm supposed to put here something regarding how this is for testing only 
  <br> xDrip & Nightscout are not official medical services or companies, <i>use at your own risk</i>
  <br> I'm not affiliated with nor do I represent either company, don't sue me, yada yada</sub></H6>

My intent is to include all the possible values you may want while giving enough information to decide if you do or not.

You can delete any lines you do not need, as I feel it much easier to delete what you don't need than it is to go searching for what you do, especially since nowhere seems to contain it all & formatting is never clearly stated anywhere.

I personally use [NginX Proxy Manager](https://hub.docker.com/r/jc21/nginx-proxy-manager), a simple, easy GUI to manage things shared externally. I have not included *any* details about setting up a Reverse Proxy, you will have to cover that on your own. Any tutorial on how to do that will work fine. Most Docker-Compose files for Nightscout I have found have included an instance of `Traefik` to do this, which *should* work fine so long as
1. You don't already have a proxy manager
2. Your router uses UPnP
3. Your server/host/computer/network doesn't already have any internal traffic on your web ports (80 & 443)
4. You're not behind a double-NAT
5. I'm sure a few other conditions

But I feel that is a separate issue & including that makes it harder to work with not easier.
For an easy setup [JC21 has a nice Docker Container](https://hub.docker.com/r/jc21/nginx-proxy-manager). Unfortunately the [instructions](https://nginxproxymanager.com/setup/#running-the-app) are not great & not included directly on DockerHub, but there are loads of YouTube Guides on it, but it's nice & doesn't require complicated setup or a separate database, plus it doesn't require settings in the docker-compose for each thing, everything to be on the same shared docker network, or elevated privileges, & has a **G**raphical **U**ser **I**nterface your Grandma could use.


## Complete docker-compose

```yaml
version: '3.9'
####    I am including most possible configuration options, they are not all necessary
####    I feel it is much easier to remove or comment in/out what you want/don't want than to go find what you need to add
####    USERNAMES & PASSWORDS are kept unique to each instance to make clear what is used in other places. Most users will use the same for many or all
####    All comments (Things after "# ") or unwanted lines can be removed
####    "/docker/cgm/nightscout/" is used in this example as the local install location. locations can be changed to suit your setup
services:
  nightscout-app:
    image: nightscout/cgm-remote-monitor:latest
    container_name: nightscout
    depends_on:
      - nightscout-mongodb
    environment:
      - PUID=1000                                               # Not necessary, but sets the USER who runs the container as not root
      - PGID=1000                                               # Not necessary, but sets the USER who runs the container as not root
      - TZ=America/Cancun                                       # Use your Time Zone from https://wikipedia.org/wiki/List_of_tz_database_time_zones
      - API_SECRET=secretAPIatLeast12Characters                 # In xDrip: REST-API > Base URL= "<<API_SECRET>>@<<BASE_URL>>/api/v1/"
      - MONGODB_URI=mongodb://nightscout01:mongoPassword01@nightscout-mongodb:27017/nightscout02
      #- HOSTNAME=0.0.0.0                                        # DEFAULT[0.0.0.0] (any)
      - INSECURE_USE_HTTP=true                                  # DEFAULT[false] - Set this to "true", unless you have your own SSL provided internally, if you wish to access without reverse Proxy
      - BASE_URL=https://me.xdrip.ext                           #URL of your Nightscout site accessed outside the local network
      - DISPLAY_UNITS=mg/dl                                     # DEFAULT[mg/dl] - CHOICES: "=mg/dl" | "=mmol" "=mmol/L" (same thing either works)
      - DEVICESTATUS_ADVANCED=true                              # required for some other plugins
      - CUSTOM_TITLE="My Nightscout"                            # website title by default
      - TIME_FORMAT=12                                          # DEFAULT[12] - CHOICES: "12" | "24"
      - THEME=colors                                            # switch to "colors" theme
      - LANGUAGE=en                                             # 
      - BASAL_RENDER=default                                    # show basal rate
      - SHOW_PLUGINS=pump openaps                               # show plugins
      - EDIT_MODE=off                                           # do not allow treatment editing
      #- AUTH_DEFAULT_ROLES=                                     # DEFAULT[readable] - CHOICES: "readable" | "denied" | <<Or any valid role name>>
    ############
      #- DISABLE="delta direction upbat" These are enabled by default, include here to disable
                 # AVAILABLE OPTIONS: "delta" (BG Delta), "direction" (BG Direction), "upbat" (Uploader Battery), "timeago" (Time Ago), "devicestatus" (Device Status)
                 # AVAILABLE OPTIONS: "errorcodes" (CGM Error Codes), "ar2" (AR2 Forecasting), "simplealarms" (Simple BG Alarms), "profile" (Treatment Profile)
    ############
      - ENABLE="careportal boluscalc food iob cob bwp cage sage iage bage basal bolus bridge googlehome speech cors dbsize" # enable optional features, expects {space delimited list} https://github.com/nightscout/cgm-remote-monitor#plugins
                 # AVAILABLE OPTIONS: careportal (Careportal), boluscalc (Bolus Wizard), food (Custom Foods), rawbg (Raw BG), iob (Insulin-on-Board), cob (Carbs-on-Board), bwp (Bolus Wizard Preview)
                 # AVAILABLE OPTIONS: cage (Cannula Age), sage (Sensor Age), iage (Insulin Age), bage (Battery Age), treatmentnotify (Treatment Notifications), basal (Basal Profile), 
                 # AVAILABLE OPTIONS: bolus (Bolus Rendering), bridge (Share2Nightscout bridge), mmconnect (MiniMed Connect bridge), pump (Pump Monitoring), openaps (OpenAPS), loop (Loop), 
                 # AVAILABLE OPTIONS: override (Override Mode), xdripjs (xDrip-js), alexa (Amazon Alexa), googlehome (Google Home/DialogFLow), speech (Speech), cors (CORS), dbsize (Database Size)
    ############
             # "upbat" (Uploader Battery) Settings
      #- UPBAT_ENABLE_ALERTS=                                    # DEFAULT[false] - (true)= enable uploader battery alarms via Pushover & IFTTT
      #- UPBAT_WARN=                                             # DEFAULT[30] - Minimum battery percent to trigger warning
      #- UPBAT_URGENT=                                           # DEFAULT[20] - Minimum battery percent to trigger urgent alarm
    ############
             # "timeago" (Time Ago) Settings
      #- TIMEAGO_ENABLE_ALERTS=                                  # DEFAULT[false] - CHOICES: true | false - (true)= enable stale data alarms via Pushover & IFTTT
      #- ALARM_TIMEAGO_WARN=                                     # DEFAULT[on] - CHOICES: on | off
      #- ALARM_TIMEAGO_WARN_MINS=                                # DEFAULT[15] - minutes since the last reading to trigger a warning
      #- ALARM_TIMEAGO_URGENT=                                   # DEFAULT[on] - CHOICES: on | off
      #- ALARM_TIMEAGO_URGENT_MINS=                              # DEFAULT[30] - minutes since the last reading to trigger a urgent alarm
    ############
             # "errorcodes" (CGM Error Codes) Settings
      #- ERRORCODES_INFO=                                        # DEFAULT["1 2 3 4 5 6 7 8"] - By default the needs calibration (blood drop) & other codes below 9 generate an info level notification, {space delimited list} of number or off to disable
      #- ERRORCODES_WARN=                                        # DEFAULT[off] - By default there are no warning configured, {space delimited list} of numbers, (off)= disable
      #- ERRORCODES_URGENT=                                      # DEFAULT[9 10] - By default the hourglass & ??? generate an urgent alarm, {space delimited list} of numbers, (off)= disable
    ############
             # "ar2" (AR2 Forecasting) Settings
      #- AR2_CONE_FACTOR=                                        # DEFAULT[2] - to adjust size of cone, use 0 for a single line
    ############
      - ALARM_TYPES="simple predict"                            # DEFAULT[simple] - CHOICES: "simple" | "predict" | ""simple predict""
             # "simplealarms" (Simple BG Alarms) Settings
      - BG_HIGH=260                                             # DEFAULT[260] - CHOICES: 40-400  - Triggers the ALARM_URGENT_HIGH alarm
      - ALARM_URGENT_HIGH=on                                    # DEFAULT[on] - CHOICES: on | off - 
      #- ALARM_URGENT_HIGH_MINS=                                     # DEFAULT["30 60 90 120"] - Number of minutes to snooze urgent high alarms
      - BG_TARGET_TOP=180                                       # DEFAULT[180] - CHOICES: 40-400  - Triggers the ALARM_HIGH alarm
      - ALARM_HIGH=on                                           # DEFAULT[on] - CHOICES: on | off - 
      #- ALARM_HIGH_MINS=                                            # DEFAULT["30 60 90 120"] - Number of minutes to snooze high alarms
      - BG_TARGET_BOTTOM=80                                     # DEFAULT[80] - CHOICES: 40-400   - Triggers the ALARM_LOW alarm
      - ALARM_LOW=on                                            # DEFAULT[on] - CHOICES: on | off - 
      #- ALARM_LOW_MINS=                                             # DEFAULT["15 30 45 60"] - Number of minutes to snooze low alarms
      - BG_LOW=55                                               # DEFAULT[55] - CHOICES: 40-400   - Triggers the ALARM_URGENT_LOW alarm
      - ALARM_URGENT_LOW=on                                     # DEFAULT[on] - CHOICES: on | off - 
      #- ALARM_URGENT_LOW_MINS=                                  # DEFAULT["15 30 45"] - Number of minutes to snooze urgent low alarms
      #- ALARM_URGENT_MIN=                                       # DEFAULT["30 60 90 120"] - Number of minutes to snooze urgent alarms (that aren't tagged as high or low)
      #- ALARM_WARN_MINS=                                        # DEFAULT["30 60 90 120"] - Number of minutes to snooze warning alarms (that aren't tagged as high or low)
    ############
             # "profile" (Treatment Profile) Settings
      #- PROFILE_HISTORY=                                        # DEFAULT[off] - CHOICES: on | off - Enable/disable NS ability to keep history of your profiles (still experimental)
      #- PROFILE_MULTIPLE=                                       # DEFAULT[off] - CHOICES: on | off - Enable/disable NS ability to handle & switch between multiple treatment profiles
    ############
    ############
            # xDrip
      #- API_SECRET=secretAPIatLeast12Characters                 # In xDrip got {Settings} > {Cloud Upload} > {Nightscout Sync (REST-API)}
      #- BASE_URL=https://me.xdrip.ext                           # {Enabled}=ON, {Base URL}= "<<API_SECRET>>@<<BASE_URL>>/api/v1/"
      #- ENABLE=“careport basal bridge”                          # e.g. secretAPIatLeast12Characters@https://me.xdrip.ext/api/v1/
    ############
            # Dexcom
      #- BRIDGE_SERVER=US                                        # DEFAULT[US] - CHOICES: "US" | "EU"
      #- BRIDGE_USER_NAME=                                       # Your Dexcom username (Same as you use to log in to the Dexcom app)
      #- BRIDGE_PASSWORD=                                        # Your Dexcom password (Same as you use to log in to the Dexcom app)
      #- ENABLE=“careport basal bridge”
    ############
            # Medtronic
      #- ENABLE=“careport basal mmconnect”
      #- MMCONNECT_USER_NAME=                                    # Your Minimed CareLink account name (Same used to log into the MiniMed Connect app)
      #- MMCONNECT_PASSWORD=                                     # Your Minimed CareLink account password (Same used to log into the MiniMed Connect app)
    ############
            # Abbott
      #- ENABLE=“careport basal”
    ############
    ############
             # Database Settings
      #- MONGO_COLLECTION=                                       # DEFAULT[entries] - Collection where CGM entries are stored
      #- MONGO_TREATMENTS_COLLECTION=                            # DEFAULT[treatments] - Collection used to store treatments entered in the Care Portal
      #- MONGO_DEVICESTATUS_COLLECTION=                          # DEFAULT[devicestatus] - Collection used to store device status information such as uploader battery
      #- MONGO_PROFILE_COLLECTION=                               # DEFAULT[profile] - Collection used to store your profiles
      #- MONGO_FOOD_COLLECTION=                                  # DEFAULT[food] - Collection used to store your food database
      #- MONGO_ACTIVITY_COLLECTION=                              # DEFAULT[activity] - Collection used to store activity data
    ports:
      - "1337:1337"
    volumes:
          ### Add custom Alert Tones ###
          # I recomend mounting to the `/opt/app/static/audio` directory, but this gives you the 4 files in the directory
          # name your files the same & place them in the mount directory or mount the files individually with whatever name
          # The original MP3 files are each ≈15 seconds, OGG files ≈34 seconds. They are all ≈30-40kbps MP3s Constant Bitrate, OGGs Variable
          # This shouldn't matter but for optimal compatibility they are good to know. I have not had problems with others personally
      #- /docker/cgm/nightscout/audio/regular.mp3:/opt/app/static/audio/alarm.mp3
      #- /docker/cgm/nightscout/audio/alarm.ogg:/opt/app/static/audio/alarm.ogg             # As far as I know these are never used
      #- /docker/cgm/nightscout/audio/urgent.mp3:/opt/app/static/audio/alarm2.mp3
      #- /docker/cgm/nightscout/audio/alarm2.ogg:/opt/app/static/audio/alarm2.ogg           # As far as I know these are never used
    #  - /docker/cgm/nightscout/audio:/opt/app/static/audio
      - /docker/log/var/log:/var/log:rw                         # As of now none of these containers use it but I always map "/var/log" for system logging
      - /etc/localtime:/etc/localtime:ro                        # Not necessary but lets the container know the system's timezone ":ro" means "Read Only"
    restart: always                                             # Sets the container to always run
    healthcheck:                                                # OPTIONAL: This adds a Healthcheck to the container. Instead of just saying `[Running]` it will say `[Healthy]` if it is unhealthy it should restart. Trying to find a better test, if it says [Unhealthy] or [Starting] while working correctly this section should be #commented out
      test: ["CMD-SHELL", "wget -q http://localhost:1337 -O - | grep 'Bolus' || (kill -s 15 1 && sleep 10 && kill -s 9 1)"]
      interval: 10s
      timeout: 10s
      retries: 3
      start_period: 20s
 ##################################################
  nightscout-mongodb:                                   # SERVICE_NAME - used by the "depends_on:" to make sure the database it running before things try to use it
    image: bitnami/mongodb:latest                       # Other options "image: mongo:latest" will need adjustments in other places such as "volumes:" mounts
    #image: docker.io/bitnami/mongodb:4.4                # I prefer to call "latest" & without "docker.io/" But others prefer to use them for stability assurance
    container_name: nightscout-mongodb                  # called in Nightscout's MONGODB_URI. General good practice to be the same as SERVICE_NAME but not required
    environment:
      - PUID=1000                                               # Refer to {nightscout-app}
      - PGID=1000                                               # Refer to {nightscout-app}
      - TZ=America/Cancun                                       # Refer to {nightscout-app}
      - MONGO_INITDB_ROOT_USERNAME=mongodbpoweruser             # May be necessry if importing from another database or doing management on the database
      - MONGO_INITDB_DATABASE=auth                              # May be optional, hard to find documaentation about it
      - MONGO_INITDB_ROOT_PASSWORD=mongoRootPassword03          # Will probably never be needed, I set it the same as MONGODB_ROOT_PASSWORD
      - MONGODB_ROOT_USER=mongodbpoweruser                      # I'm not sure what the differences are but is usually set the same as MONGO_INITDB_ROOT_USERNAME
      - MONGODB_ROOT_PASSWORD=mongoaRootPassword02              # I'm not sure what the differences are but is usually set the same as MONGO_INITDB_ROOT_PASSWORD
      - MONGODB_USERNAME=nightscout01                           # Username called in Nightscout's MONGODB_URI - I set to just "Nightscout"
      - MONGODB_PASSWORD=mongoPassword01                        # Password called in Nightscout's MONGODB_URI
      - MONGODB_DATABASE=nightscout02                           # Database called in Nightscout's MONGODB_URI - I set to just "Nightscout"
      - MONGODB_PORT_NUMBER=27017                               # DEFAULT[27017] - Port each MongoDB will use inernally 27017:<<MONGODB_PORT_NUMBER>>
    #ports:                                                  # Only needed to access database externally. If all parts are in the same stack/network removal or # suggested
    #  - 27017:27017                                             # DEFAULT[27017:27017] - <<EXTERNAL PORT>>:<<MONGODB_PORT_NUMBER>>
    volumes:
      - /docker/cgm/nightscout/database:/bitnami/mongodb        # /LOCATION/ON/YOUR/SYSTEM:/CONTAINER/LOCATION - for "mongo:latest" use ":/data/db"
      - /docker/log/var/log:/var/log:rw                         # Refer to {nightscout-app}
      - /etc/localtime:/etc/localtime:ro                        # Refer to {nightscout-app}
    restart: always                                             # Refer to {nightscout-app}
    healthcheck:                                                # OPTIONAL: This adds a Healthcheck to the container. Instead of just saying `[Running]` it will say `[Healthy]` if it is unhealthy it should restart
      test: ["CMD", "sh", "-c", "mongosh --eval 'db.runCommand(\"ping\")' || exit 1", "||", "bash", "-c", "kill -s 15 1 && sleep 10 && kill -s 9 1"]
      interval: 10s
      timeout: 10s
      retries: 3
      start_period: 20s
 ##################################################
  nightscout-mongo-express:
    image: mongo-express
    container_name: nightscout-mongo-express
    depends_on:
      - nightscout-mongodb                                      # This means this container will not start until the Database Container starts
    environment:
      - PUID=1000                                               # Refer to {nightscout-app}
      - PGID=1000                                               # Refer to {nightscout-app}
      - TZ=America/Cancun                                       # Refer to {nightscout-app}
      - ME_CONFIG_MONGODB_ADMINUSERNAME=mongodbpoweruser        # <<MONGODB_ROOT_USER>> in {nightscout-mongodb} - used to manage the Database
      - ME_CONFIG_MONGODB_ADMINPASSWORD=mongoaRootPassword02    # <<MONGODB_ROOT_PASSWORD>> in {nightscout-mongodb} - used to manage the Database
      - ME_CONFIG_MONGODB_URL=mongodb://mongodbpoweruser:mongoaRootPassword02@nightscout-mongodb:27017/ # mongodb://<<MONGODB_ROOT_USER>>:<<MONGODB_ROOT_PASSWORD>>@<<CONTAINER_NAME>>:<<MONGODB_PORT_NUMBER>>/
    ports:
      - 18081:8081                                              # DEFAULT[8081:8081] - Management Port for MongoDB="http://YOUR.IP.ADD.RESS:18081" - Internally="nightscout-mongo-express:8081"
    volumes:
      - /docker/log/var/log:/var/log:rw                         # Refer to {nightscout-app}
      - /etc/localtime:/etc/localtime:ro                        # Refer to {nightscout-app}
    restart: always                                             # Refer to {nightscout-app}
    healthcheck:                                                # OPTIONAL: This adds a Healthcheck to the container. Instead of just saying `[Running]` it will say `[Healthy]` if it is unhealthy it will restart
      test: ["CMD-SHELL", "wget -q http://localhost:8081 -O - | grep 'Mongo Express' || exit 1", "||", "bash", "-c", "kill -s 15 1 && sleep 10 && kill -s 9 1"]
      interval: 10s
      timeout: 10s
      retries: 3
      start_period: 20s

```


## Easy docker-compose for xDrip

```yaml
version: '3.9'
####    I am including most possible configuration options, they are not all necessary
####    I feel it is much easier to remove or comment in/out what you want/don't want than to go find what you need to add
####    USERNAMES & PASSWORDS are kept unique to each instance to make clear what is used in other places. Most users will use the same for many or all
####    All comments (Things after "# ") or unwanted lines can be removed
####    "/docker/cgm/nightscout/" is used in this example as the local install location. locations can be changed to suit your setup
services:
  nightscout-app:
    image: nightscout/cgm-remote-monitor:latest
    container_name: nightscout
    depends_on:
      - nightscout-mongodb
    environment:
      - PUID=1000                                               # Not necessary, but sets the USER who runs the container as not root
      - PGID=1000                                               # Not necessary, but sets the USER who runs the container as not root
      - TZ=America/Cancun                                       # Use your Time Zone from https://wikipedia.org/wiki/List_of_tz_database_time_zones
      - API_SECRET=secretAPIatLeast12Characters                 # In xDrip: REST-API > Base URL= "<<API_SECRET>>@<<BASE_URL>>/api/v1/"
      - MONGODB_URI=mongodb://nightscout01:mongoPassword01@nightscout-mongodb:27017/nightscout02
      - INSECURE_USE_HTTP=true                                  # DEFAULT[false] - Set this to "true", unless you have your own SSL provided internally, if you wish to access without reverse Proxy
      - BASE_URL=https://me.xdrip.ext                           #URL of your Nightscout site accessed outside the local network
      - DISPLAY_UNITS=mg/dl                                     # DEFAULT[mg/dl] - CHOICES: "=mg/dl" | "=mmol" "=mmol/L" (same thing either works)
      - DEVICESTATUS_ADVANCED=true                              # required for some other plugins
      - CUSTOM_TITLE="My Nightscout"                            # website title by default
      - TIME_FORMAT=12                                          # DEFAULT[12] - CHOICES: "12" | "24"
      - THEME=colors                                            # switch to "colors" theme
      - LANGUAGE=en                                             # 
      - BASAL_RENDER=default                                    # show basal rate
      - SHOW_PLUGINS=pump openaps                               # show plugins
      - EDIT_MODE=off                                           # do not allow treatment editing
      #- AUTH_DEFAULT_ROLES=                                     # DEFAULT[readable] - CHOICES: "readable" | "denied" | <<Or any valid role name>>
    ############
      - ENABLE="careport basal bridge"                           # enable optional features, expects {space delimited list} https://github.com/nightscout/cgm-remote-monitor#plugins
    ############
             # "upbat" (Uploader Battery) Settings
      #- UPBAT_ENABLE_ALERTS=                                    # DEFAULT[false] - (true)= enable uploader battery alarms via Pushover & IFTTT
      #- UPBAT_WARN=                                             # DEFAULT[30] - Minimum battery percent to trigger warning
      #- UPBAT_URGENT=                                           # DEFAULT[20] - Minimum battery percent to trigger urgent alarm
    ############
             # "timeago" (Time Ago) Settings
      #- TIMEAGO_ENABLE_ALERTS=                                  # DEFAULT[false] - CHOICES: true | false - (true)= enable stale data alarms via Pushover & IFTTT
      #- ALARM_TIMEAGO_WARN=                                     # DEFAULT[on] - CHOICES: on | off
      #- ALARM_TIMEAGO_WARN_MINS=                                # DEFAULT[15] - minutes since the last reading to trigger a warning
      #- ALARM_TIMEAGO_URGENT=                                   # DEFAULT[on] - CHOICES: on | off
      #- ALARM_TIMEAGO_URGENT_MINS=                              # DEFAULT[30] - minutes since the last reading to trigger a urgent alarm
    ############
      - ALARM_TYPES="simple"                                    # DEFAULT[simple] - CHOICES: "simple" | "predict" | ""simple predict""
             # "simplealarms" (Simple BG Alarms) Settings
      - BG_HIGH=260                                             # DEFAULT[260] - CHOICES: 40-400  - Triggers the ALARM_URGENT_HIGH alarm
      - ALARM_URGENT_HIGH=on                                    # DEFAULT[on] - CHOICES: on | off - 
      #- ALARM_URGENT_HIGH_MINS=                                     # DEFAULT["30 60 90 120"] - Number of minutes to snooze urgent high alarms
      - BG_TARGET_TOP=190                                       # DEFAULT[180] - CHOICES: 40-400  - Triggers the ALARM_HIGH alarm
      - ALARM_HIGH=on                                           # DEFAULT[on] - CHOICES: on | off - 
      #- ALARM_HIGH_MINS=                                            # DEFAULT["30 60 90 120"] - Number of minutes to snooze high alarms
      - BG_TARGET_BOTTOM=80                                     # DEFAULT[80] - CHOICES: 40-400   - Triggers the ALARM_LOW alarm
      - ALARM_LOW=on                                            # DEFAULT[on] - CHOICES: on | off - 
      #- ALARM_LOW_MINS=                                             # DEFAULT["15 30 45 60"] - Number of minutes to snooze low alarms
      - BG_LOW=55                                               # DEFAULT[55] - CHOICES: 40-400   - Triggers the ALARM_URGENT_LOW alarm
      - ALARM_URGENT_LOW=on                                     # DEFAULT[on] - CHOICES: on | off - 
      #- ALARM_URGENT_LOW_MINS=                                  # DEFAULT["15 30 45"] - Number of minutes to snooze urgent low alarms
      #- ALARM_URGENT_MIN=                                       # DEFAULT["30 60 90 120"] - Number of minutes to snooze urgent alarms (that aren't tagged as high or low)
      #- ALARM_WARN_MINS=                                        # DEFAULT["30 60 90 120"] - Number of minutes to snooze warning alarms (that aren't tagged as high or low)
    ############
    ############
            # xDrip
      #- API_SECRET=secretAPIatLeast12Characters                 # In xDrip got {Settings} > {Cloud Upload} > {Nightscout Sync (REST-API)}
      #- BASE_URL=https://me.xdrip.ext                           # {Enabled}=ON, {Base URL}= "<<API_SECRET>>@<<BASE_URL>>/api/v1/"
      #- ENABLE=“careport basal bridge”
    ports:
      - "1337:1337"
    volumes:
      #- /docker/cgm/nightscout/audio:/opt/app/static/audio
      - /docker/log/var/log:/var/log:rw                         # As of now none of these containers use it but I always map "/var/log" for system logging
      - /etc/localtime:/etc/localtime:ro                        # Not necessary but lets the container know the system's timezone ":ro" means "Read Only"
    restart: always                                             # Sets the container to always run
    healthcheck:                                                # OPTIONAL: This adds a Healthcheck to the container. Instead of just saying `[Running]` it will say `[Healthy]` if it is unhealthy it should restart. Trying to find a better test, if it says [Unhealthy] or [Starting] while working correctly this section should be #commented out
      test: ["CMD-SHELL", "wget -q http://localhost:1337 -O - | grep 'Bolus' || (kill -s 15 1 && sleep 10 && kill -s 9 1)"]
      interval: 10s
      timeout: 10s
      retries: 3
      start_period: 20s
 ##################################################
  nightscout-mongodb:                                   # SERVICE_NAME - used by the "depends_on:" to make sure the database it running before things try to use it
    image: bitnami/mongodb:latest                       # Other options "image: mongo:latest" will need adjustments in other places such as "volumes:" mounts
    #image: docker.io/bitnami/mongodb:4.4                # I prefer to call "latest" & without "docker.io/" But others prefer to use them for stability assurance
    container_name: nightscout-mongodb                  # called in Nightscout's MONGODB_URI. General good practice to be the same as SERVICE_NAME but not required
    environment:
      - PUID=1000                                               # Refer to {nightscout-app}
      - PGID=1000                                               # Refer to {nightscout-app}
      - TZ=America/Cancun                                       # Refer to {nightscout-app}
      - MONGO_INITDB_ROOT_USERNAME=mongodbpoweruser             # May be necessry if importing from another database or doing management on the database
      - MONGO_INITDB_DATABASE=auth                              # May be optional, hard to find documaentation about it
      - MONGO_INITDB_ROOT_PASSWORD=mongoRootPassword03          # Will probably never be needed, I set it the same as MONGODB_ROOT_PASSWORD
      - MONGODB_ROOT_USER=mongodbpoweruser                      # I'm not sure what the differences are but is usually set the same as MONGO_INITDB_ROOT_USERNAME
      - MONGODB_ROOT_PASSWORD=mongoaRootPassword02              # I'm not sure what the differences are but is usually set the same as MONGO_INITDB_ROOT_PASSWORD
      - MONGODB_USERNAME=nightscout01                           # Username called in Nightscout's MONGODB_URI - I set to just "Nightscout"
      - MONGODB_PASSWORD=mongoPassword01                        # Password called in Nightscout's MONGODB_URI
      - MONGODB_DATABASE=nightscout02                           # Database called in Nightscout's MONGODB_URI - I set to just "Nightscout"
      - MONGODB_PORT_NUMBER=27017                               # DEFAULT[27017] - Port each MongoDB will use inernally 27017:<<MONGODB_PORT_NUMBER>>
    #ports:                                                  # Only needed to access database externally. If all parts are in the same stack/network removal or # suggested
    #  - 27017:27017                                             # DEFAULT[27017:27017] - <<EXTERNAL PORT>>:<<MONGODB_PORT_NUMBER>>
    volumes:
      - /docker/cgm/nightscout/database:/bitnami/mongodb        # /LOCATION/ON/YOUR/SYSTEM:/CONTAINER/LOCATION - for "mongo:latest" use ":/data/db"
      - /docker/log/var/log:/var/log:rw                         # Refer to {nightscout-app}
      - /etc/localtime:/etc/localtime:ro                        # Refer to {nightscout-app}
    restart: always                                             # Refer to {nightscout-app}
    healthcheck:                                                # OPTIONAL: This adds a Healthcheck to the container. Instead of just saying `[Running]` it will say `[Healthy]` if it is unhealthy it should restart
      test: ["CMD", "sh", "-c", "mongosh --eval 'db.runCommand(\"ping\")' || exit 1", "||", "bash", "-c", "kill -s 15 1 && sleep 10 && kill -s 9 1"]
      interval: 10s
      timeout: 10s
      retries: 3
      start_period: 20s
 ##################################################
  nightscout-mongo-express:
    image: mongo-express
    container_name: nightscout-mongo-express
    depends_on:
      - nightscout-mongodb                                      # This means this container will not start until the Database Container starts
    environment:
      - PUID=1000                                               # Refer to {nightscout-app}
      - PGID=1000                                               # Refer to {nightscout-app}
      - TZ=America/Cancun                                       # Refer to {nightscout-app}
      - ME_CONFIG_MONGODB_ADMINUSERNAME=mongodbpoweruser        # <<MONGODB_ROOT_USER>> in {nightscout-mongodb} - used to manage the Database
      - ME_CONFIG_MONGODB_ADMINPASSWORD=mongoaRootPassword02    # <<MONGODB_ROOT_PASSWORD>> in {nightscout-mongodb} - used to manage the Database
      - ME_CONFIG_MONGODB_URL=mongodb://mongodbpoweruser:mongoaRootPassword02@nightscout-mongodb:27017/ # mongodb://<<MONGODB_ROOT_USER>>:<<MONGODB_ROOT_PASSWORD>>@<<CONTAINER_NAME>>:<<MONGODB_PORT_NUMBER>>/
    ports:
      - 18081:8081                                              # DEFAULT[8081:8081] - Management Port for MongoDB="http://YOUR.IP.ADD.RESS:18081" - Internally="nightscout-mongo-express:8081"
    volumes:
      - /docker/log/var/log:/var/log:rw                         # Refer to {nightscout-app}
      - /etc/localtime:/etc/localtime:ro                        # Refer to {nightscout-app}
    restart: always                                             # Refer to {nightscout-app}
    healthcheck:                                                # OPTIONAL: This adds a Healthcheck to the container. Instead of just saying `[Running]` it will say `[Healthy]` if it is unhealthy it will restart
      test: ["CMD-SHELL", "wget -q http://localhost:8081 -O - | grep 'Mongo Express' || exit 1", "||", "bash", "-c", "kill -s 15 1 && sleep 10 && kill -s 9 1"]
      interval: 10s
      timeout: 10s
      retries: 3
      start_period: 20s

```


## Easy docker-compose for Dexcom

```yaml
version: '3.9'
####    I am including most possible configuration options, they are not all necessary
####    I feel it is much easier to remove or comment in/out what you want/don't want than to go find what you need to add
####    USERNAMES & PASSWORDS are kept unique to each instance to make clear what is used in other places. Most users will use the same for many or all
####    All comments (Things after "# ") or unwanted lines can be removed
####    "/docker/cgm/nightscout/" is used in this example as the local install location. locations can be changed to suit your setup
services:
  nightscout-app:
    image: nightscout/cgm-remote-monitor:latest
    container_name: nightscout
    depends_on:
      - nightscout-mongodb
    environment:
      - PUID=1000                                               # Not necessary, but sets the USER who runs the container as not root
      - PGID=1000                                               # Not necessary, but sets the USER who runs the container as not root
      - TZ=America/Cancun                                       # Use your Time Zone from https://wikipedia.org/wiki/List_of_tz_database_time_zones
      - API_SECRET=secretAPIatLeast12Characters                 # In xDrip: REST-API > Base URL= "<<API_SECRET>>@<<BASE_URL>>/api/v1/"
      - MONGODB_URI=mongodb://nightscout01:mongoPassword01@nightscout-mongodb:27017/nightscout02
      #- HOSTNAME=0.0.0.0                                        # DEFAULT[0.0.0.0] (any)
      - INSECURE_USE_HTTP=true                                  # DEFAULT[false] - Set this to "true", unless you have your own SSL provided internally, if you wish to access without reverse Proxy
      - BASE_URL=https://me.xdrip.ext                           #URL of your Nightscout site accessed outside the local network
      - DISPLAY_UNITS=mg/dl                                     # DEFAULT[mg/dl] - CHOICES: "=mg/dl" | "=mmol" "=mmol/L" (same thing either works)
      - DEVICESTATUS_ADVANCED=true                              # required for some other plugins
      - CUSTOM_TITLE="My Nightscout"                            # website title by default
      - TIME_FORMAT=12                                          # DEFAULT[12] - CHOICES: "12" | "24"
      - THEME=colors                                            # switch to "colors" theme
      - LANGUAGE=en                                             # 
      - BASAL_RENDER=default                                    # show basal rate
      - SHOW_PLUGINS=pump openaps                               # show plugins
      - EDIT_MODE=off                                           # do not allow treatment editing
      #- AUTH_DEFAULT_ROLES=                                     # DEFAULT[readable] - CHOICES: "readable" | "denied" | <<Or any valid role name>>
    ############
      - ENABLE="careport basal bridge"                          # enable optional features, expects {space delimited list} https://github.com/nightscout/cgm-remote-monitor#plugins
    ############
             # "timeago" (Time Ago) Settings
      #- TIMEAGO_ENABLE_ALERTS=                                  # DEFAULT[false] - CHOICES: true | false - (true)= enable stale data alarms via Pushover & IFTTT
      #- ALARM_TIMEAGO_WARN=                                     # DEFAULT[on] - CHOICES: on | off
      #- ALARM_TIMEAGO_WARN_MINS=                                # DEFAULT[15] - minutes since the last reading to trigger a warning
      #- ALARM_TIMEAGO_URGENT=                                   # DEFAULT[on] - CHOICES: on | off
      #- ALARM_TIMEAGO_URGENT_MINS=                              # DEFAULT[30] - minutes since the last reading to trigger a urgent alarm
    ############
      - ALARM_TYPES="simple"                                    # DEFAULT[simple] - CHOICES: "simple" | "predict" | ""simple predict""
             # "simplealarms" (Simple BG Alarms) Settings
      - BG_HIGH=260                                             # DEFAULT[260] - CHOICES: 40-400  - Triggers the ALARM_URGENT_HIGH alarm
      - ALARM_URGENT_HIGH=on                                    # DEFAULT[on] - CHOICES: on | off - 
      #- ALARM_URGENT_HIGH_MINS=                                     # DEFAULT["30 60 90 120"] - Number of minutes to snooze urgent high alarms
      - BG_TARGET_TOP=180                                       # DEFAULT[180] - CHOICES: 40-400  - Triggers the ALARM_HIGH alarm
      - ALARM_HIGH=on                                           # DEFAULT[on] - CHOICES: on | off - 
      #- ALARM_HIGH_MINS=                                            # DEFAULT["30 60 90 120"] - Number of minutes to snooze high alarms
      - BG_TARGET_BOTTOM=80                                     # DEFAULT[80] - CHOICES: 40-400   - Triggers the ALARM_LOW alarm
      - ALARM_LOW=on                                            # DEFAULT[on] - CHOICES: on | off - 
      #- ALARM_LOW_MINS=                                             # DEFAULT["15 30 45 60"] - Number of minutes to snooze low alarms
      - BG_LOW=55                                               # DEFAULT[55] - CHOICES: 40-400   - Triggers the ALARM_URGENT_LOW alarm
      - ALARM_URGENT_LOW=on                                     # DEFAULT[on] - CHOICES: on | off - 
      #- ALARM_URGENT_LOW_MINS=                                  # DEFAULT["15 30 45"] - Number of minutes to snooze urgent low alarms
      #- ALARM_URGENT_MIN=                                       # DEFAULT["30 60 90 120"] - Number of minutes to snooze urgent alarms (that aren't tagged as high or low)
      #- ALARM_WARN_MINS=                                        # DEFAULT["30 60 90 120"] - Number of minutes to snooze warning alarms (that aren't tagged as high or low)
    ############
    ############
            # Dexcom
      #- BRIDGE_SERVER=US                                        # DEFAULT[US] - CHOICES: "US" | "EU"
      #- BRIDGE_USER_NAME=                                       # Your Dexcom username (Same as you use to log in to the Dexcom app)
      #- BRIDGE_PASSWORD=                                        # Your Dexcom password (Same as you use to log in to the Dexcom app)
      #- ENABLE=“careport basal bridge”
    ports:
      - "1337:1337"
    volumes:
      #- /docker/cgm/nightscout/audio:/opt/app/static/audio
      - /docker/log/var/log:/var/log:rw                         # As of now none of these containers use it but I always map "/var/log" for system logging
      - /etc/localtime:/etc/localtime:ro                        # Not necessary but lets the container know the system's timezone ":ro" means "Read Only"
    restart: always                                             # Sets the container to always run
    healthcheck:                                                # OPTIONAL: This adds a Healthcheck to the container. Instead of just saying `[Running]` it will say `[Healthy]` if it is unhealthy it should restart. Trying to find a better test, if it says [Unhealthy] or [Starting] while working correctly this section should be #commented out
      test: ["CMD-SHELL", "wget -q http://localhost:1337 -O - | grep 'Bolus' || (kill -s 15 1 && sleep 10 && kill -s 9 1)"]
      interval: 10s
      timeout: 10s
      retries: 3
      start_period: 20s
 ##################################################
  nightscout-mongodb:                                   # SERVICE_NAME - used by the "depends_on:" to make sure the database it running before things try to use it
    image: bitnami/mongodb:latest                       # Other options "image: mongo:latest" will need adjustments in other places such as "volumes:" mounts
    #image: docker.io/bitnami/mongodb:4.4                # I prefer to call "latest" & without "docker.io/" But others prefer to use them for stability assurance
    container_name: nightscout-mongodb                  # called in Nightscout's MONGODB_URI. General good practice to be the same as SERVICE_NAME but not required
    environment:
      - PUID=1000                                               # Refer to {nightscout-app}
      - PGID=1000                                               # Refer to {nightscout-app}
      - TZ=America/Cancun                                       # Refer to {nightscout-app}
      - MONGO_INITDB_ROOT_USERNAME=mongodbpoweruser             # May be necessry if importing from another database or doing management on the database
      - MONGO_INITDB_DATABASE=auth                              # May be optional, hard to find documaentation about it
      - MONGO_INITDB_ROOT_PASSWORD=mongoRootPassword03          # Will probably never be needed, I set it the same as MONGODB_ROOT_PASSWORD
      - MONGODB_ROOT_USER=mongodbpoweruser                      # I'm not sure what the differences are but is usually set the same as MONGO_INITDB_ROOT_USERNAME
      - MONGODB_ROOT_PASSWORD=mongoaRootPassword02              # I'm not sure what the differences are but is usually set the same as MONGO_INITDB_ROOT_PASSWORD
      - MONGODB_USERNAME=nightscout01                           # Username called in Nightscout's MONGODB_URI - I set to just "Nightscout"
      - MONGODB_PASSWORD=mongoPassword01                        # Password called in Nightscout's MONGODB_URI
      - MONGODB_DATABASE=nightscout02                           # Database called in Nightscout's MONGODB_URI - I set to just "Nightscout"
      - MONGODB_PORT_NUMBER=27017                               # DEFAULT[27017] - Port each MongoDB will use inernally 27017:<<MONGODB_PORT_NUMBER>>
    #ports:                                                  # Only needed to access database externally. If all parts are in the same stack/network removal or # suggested
    #  - 27017:27017                                             # DEFAULT[27017:27017] - <<EXTERNAL PORT>>:<<MONGODB_PORT_NUMBER>>
    volumes:
      - /docker/cgm/nightscout/database:/bitnami/mongodb        # /LOCATION/ON/YOUR/SYSTEM:/CONTAINER/LOCATION - for "mongo:latest" use ":/data/db"
      - /docker/log/var/log:/var/log:rw                         # Refer to {nightscout-app}
      - /etc/localtime:/etc/localtime:ro                        # Refer to {nightscout-app}
    restart: always                                             # Refer to {nightscout-app}
    healthcheck:                                                # OPTIONAL: This adds a Healthcheck to the container. Instead of just saying `[Running]` it will say `[Healthy]` if it is unhealthy it should restart
      test: ["CMD", "sh", "-c", "mongosh --eval 'db.runCommand(\"ping\")' || exit 1", "||", "bash", "-c", "kill -s 15 1 && sleep 10 && kill -s 9 1"]
      interval: 10s
      timeout: 10s
      retries: 3
      start_period: 20s
 ##################################################
  nightscout-mongo-express:
    image: mongo-express
    container_name: nightscout-mongo-express
    depends_on:
      - nightscout-mongodb                                      # This means this container will not start until the Database Container starts
    environment:
      - PUID=1000                                               # Refer to {nightscout-app}
      - PGID=1000                                               # Refer to {nightscout-app}
      - TZ=America/Cancun                                       # Refer to {nightscout-app}
      - ME_CONFIG_MONGODB_ADMINUSERNAME=mongodbpoweruser        # <<MONGODB_ROOT_USER>> in {nightscout-mongodb} - used to manage the Database
      - ME_CONFIG_MONGODB_ADMINPASSWORD=mongoaRootPassword02    # <<MONGODB_ROOT_PASSWORD>> in {nightscout-mongodb} - used to manage the Database
      - ME_CONFIG_MONGODB_URL=mongodb://mongodbpoweruser:mongoaRootPassword02@nightscout-mongodb:27017/ # mongodb://<<MONGODB_ROOT_USER>>:<<MONGODB_ROOT_PASSWORD>>@<<CONTAINER_NAME>>:<<MONGODB_PORT_NUMBER>>/
    ports:
      - 8081:8081                                              # DEFAULT[8081:8081] - Management Port for MongoDB="http://YOUR.IP.ADD.RESS:18081" - Internally="nightscout-mongo-express:8081"
    volumes:
      - /docker/log/var/log:/var/log:rw                         # Refer to {nightscout-app}
      - /etc/localtime:/etc/localtime:ro                        # Refer to {nightscout-app}
    restart: always                                             # Refer to {nightscout-app}
    healthcheck:                                                # OPTIONAL: This adds a Healthcheck to the container. Instead of just saying `[Running]` it will say `[Healthy]` if it is unhealthy it will restart
      test: ["CMD-SHELL", "wget -q http://localhost:8081 -O - | grep 'Mongo Express' || exit 1", "||", "bash", "-c", "kill -s 15 1 && sleep 10 && kill -s 9 1"]
      interval: 10s
      timeout: 10s
      retries: 3
      start_period: 20s

```



