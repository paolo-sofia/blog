### Create a new site
```sh
docker run --rm -v /home/ubuntu/git/blog:/project ghcr.io/gohugoio/hugo:latest new site blog --format yaml
```

### Create a new page
```sh
docker run --rm -v /home/ubuntu/git/blog/blog:/project ghcr.io/gohugoio/hugo:latest new content posts/test-docker.md
```

### After deploy, always restart the container
```sh
docker compose restart
```
