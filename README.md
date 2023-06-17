# Blog dcavalei.com

## dev
I am using docker compose for development. This is not how you deploy the blog.
```bash
docker compose up
docker compose down
```
Running on -> http://localhost:1313/

### References
- hugo - https://gohugo.io/
- docker compose - https://docs.docker.com/compose/

### Troubleshooting
You need to clone this repo with the `--recursive` flag, I'm using submodules
```bash
git clone --recursive git@github.com:dcavalei/blog.git
```
