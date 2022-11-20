```
- docker image prune
- docker rmi $(docker images -f "dangling=true" -q)
- docker inspect --format='{{.Id}} {{.Parent}}' $(docker images --filter since=<image_id> -q)
```

-   `docker images` — List all the Docker images located on the system
-   `-f "dangling=true"` — Filter ( `-f`) the list of images to only return dangling images.
-   `-q` — Quiet output → Only display the image IDs of the Docker images

`docker rmi $INPUT` :

-   `docker rmi` — Remove Docker images (**R**e**M**ove **I**mages) by their image Id or name. You can provide multiple values.
-   `$INPUT` — The list of image IDs that the above `docker images` command returns.