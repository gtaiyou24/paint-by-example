# paint-by-example

## How to

<details><summary>setup project</summary>

```bash
docker build -t paint-by-example:development . -f ./Dockerfile.development
```
</details>

<details><summary>format</summary>

```bash
docker container run --rm -it \
    -v `pwd`:/app \
    -p 5173:5173 \
    paint-by-example:development format
```
</details>

<details><summary>compile and hot-reload for development</summary>

```sh
docker container run --rm -it \
    -v `pwd`:/app \
    -p 5173:5173 \
    paint-by-example:development dev
```
</details>

<details><summary>lint with ESLint</summary>

[ESLint](https://eslint.org/)

```bash
docker container run --rm -it \
    -v `pwd`:/app \
    -p 5173:5173 \
    paint-by-example:development lint
```
</details>

<details><summary>type-check, compile and minify for production</summary>

```sh
docker build -t paint-by-example:latest .
docker container run --rm -it \
    -v `pwd`:/app \
    -p 8080:80 \
    paint-by-example:latest
```
</details>

