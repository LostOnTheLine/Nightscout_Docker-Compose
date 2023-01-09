# Nightscout_Docker-Compose
A comprehensive Docker-Compose for a self-hosted Nightscout CGM (Glucose Monitoring) service
<H6 align="right"><sub>I think I'm supposed to put here something regarding how this is for testing only 
  <br> xDrip & Nightscout are not official medical services or companies, <i>use at your own risk</i>
  <br> I'm not affiliated with nor do I represent either company, don't sue me, yada yada</sub></H6>

My intent is to include all the possible values you may want while giving enough information to decide if you do or not.

You can delete any lines you do not need, as I feel it much easier to delete what you don't need than it is to go searching for what you do, especially since nowhere seems to contain it all & formatting is never clearly stated anywhere.

I personally use [NginX Proxy Manager](https://hub.docker.com/r/jc21/nginx-proxy-manager), a simple, easy GUI to manage things shared externally. I have not included *any* details about setting up a Reverse Proxy, you will have to cover that on your own. Any tutorial on how to do that will work fine. Most Docker-Compose files I have found have included an instance of `Traefik` to do this, which should work fine so long as
1. you don't already have a proxy manager
2. your router uses UPnP
3. your server/host/computer/network doesn't already any internal traffic on your web ports (80 & 443)
4. you're not behind a double-NAT
5. & I'm sure a few other conditions

But I feel that is a separate issue & including that makes it harder to work with not easier.
For an easy setup [JC21](https://hub.docker.com/r/jc21/nginx-proxy-manager) has a nice Docker Container. Unfortunately the [instructions](https://nginxproxymanager.com/setup/#running-the-app) are not great & not included directly on DockerHub, but there are loads of YouTube Guides on it, but it's nice & doesn't require complicated setup or a separate database, plus it doesn't require settings in the docker-compose for each thing, everything to be on the same shared docker network, or elevated privileges, & has a Graphical interface your Grandma could use.


## Easy docker-compose

```yaml


```


## Complete docker-compose

```yaml


```


## xDrip docker-compose

```yaml


```


## Dexcom docker-compose

```yaml


```



