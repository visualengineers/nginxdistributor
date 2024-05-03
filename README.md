# Sources

https://mindsers.blog/en/post/https-using-nginx-certbot-docker/
https://dev.to/isthatcentered/testing-an-angular-build-with-base-href-locally-44af
https://gist.github.com/soheilhy/8b94347ff8336d971ad0

# Important Commands

Start the environment

```bash
docker compose up
```

Update the Environment

```bash
docker compose restart
```

If you want to avoid service interruptions (even for a couple of seconds) reload it inside the container using:

```bash
docker compose exec webserver nginx -s reload
```