#### docker registry config repo
##### concept:
a private storage, where all our docker images are stored and managed.
<br>
<h4>why use it? why not just any storage?</h4>
- docker manages push and pull with smart caching, moving only necessary layers of data. 
- easy setup - with docker compose, you just need to change the image name, and that's it! you got the new version.
- build in security and access management.

##### how to create and setup?
- setup auth with [apache htpasswd](https://httpd.apache.org/docs/2.4/programs/htpasswd.html), and get the auth file, after setting users and passwords.
to setup auth: 
<ol>
<li>setup 3 env vars: REGISTRY_AUTH(auth provider, htpasswd), REGISTRY_AUTH_HTPASSWD_PATH(auth file path), REGISTRY_AUTH_HTPASSWD_REALM(which is Registry realm)</li>
<li>add the auth file as a bind mount volume, so it can reflect any changes made in the host machine to the container</li>
</ol>
- setup TLS for secure connection. you can use letsencrypt or any other service for that. <br>
to do that: 
<ol>
<li>use the following env vars: REGISTRY_HTTP_TLS_CERTIFICATE(fullchain.pem file), REGISTRY_HTTP_TLS_KEY(private key pem), and LETSENCRYPT_HOST, LETSENCRYPT_EMAIL</li>
<li>add the cert folder location as bind mount, same as with the auth. </li>
</ol>
- make sure the volumes paths are correct, then run `docker compose up -d`.

##### how to use? 
- run(only once!) `docker login <server dns/ip:port>` and type user and password, for example: `docker login exapmle-server.com:5000`.
- when pushing an image, you need to tag it with a prefix of the server name, so it would look like this: `<server name:port/image name:image tag>`
for example: docker tag example-program:1.0.0 server_addr.com:5000/example-program:1.0.0. after that, just push the new image name, and it will be in the registry.

##### notes:
- make sure the tls certificates are valid! so far, I had an issue with soft links in bind mount volumes, so I had to manually set the tls certs every 3 month. if you have the time, find a better solution. 
- it doesn't have to be dns name, the registry server could be an IP address as well.
- the storage volume is persistent. plan ahead and use enough space. 
- access management - you may want to  basic auth providers don't give you that option. to do that, read the next section:

##### setup access management - tips
- why you need it? right now, we have a user(admin), with rights to do anything. it's not ideal, especially if we send NUCs or rocks to users, they will have admin access to our docker registry, and it's a huge security risk.
- the way is to create an authorization server, which can give minimal access to users according to their needs. 
<br> it will take about two weeks to develop, for an experienced dev. an easier way is to use [something ready](https://github.com/cesanta/docker_auth) <br>
- an example with [nginx](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-docker-registry-on-ubuntu-14-04#step-four-%E2%80%94-secure-your-docker-registry-with-nginx)
- an even better way - just move to a managed registry, like dockerhub, jfrog, or gitlab. they'll manage this for you.
