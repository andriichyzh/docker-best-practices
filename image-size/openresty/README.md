
### OpenResty

```bash
cd image-size/openresty/

docker build -t "openresty:regular" regular
docker build -t "openresty:optimization" optimization
docker build -t "openresty:multistage" multistage

docker images
```
