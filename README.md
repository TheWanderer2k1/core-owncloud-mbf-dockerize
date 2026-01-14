### Init database
docker run -d \
  --name mobidrive-postgres \
  -e POSTGRES_DB=mobidrive \
  -e POSTGRES_USER=hoanganh \
  -e POSTGRES_PASSWORD=hoanganh \
  -p 5432:5432 \
  postgres:16-alpine

docker exec -it mobidrive-postgres psql -U hoanganh -d mobidrive -c "GRANT ALL ON SCHEMA public TO hoanganh; GRANT ALL ON SCHEMA public TO public; ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES TO hoanganh; ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON SEQUENCES TO hoanganh;"

### Clone code
git clone https://github.com/TheWanderer2k1/core-owncloud-mbf.git owncloud

### Build from src
docker compose run --rm build-owncloud

### Run owncloud
docker compose up owncloud

### Run all in one dockerfile
docker build \
--build-arg OWNCLOUD_DOMAIN=<domain dùng để truy cập trên trình duyệt> \
--build-arg OWNCLOUD_IP=<ip dùng để truy cập trên trình duyệt> \
--build-arg DB_TYPE=<mysql/pgsql> \
--build-arg DB_NAME=<ten db đã tạo> \
--build-arg DB_USER=<tên user db đã tạo> \
--build-arg DB_PASS=<pass của user> \
--build-arg DB_HOST=<host của db> \
--build-arg DB_PORT=<port của db> \
--build-arg ADMIN_USER=<username admin mặc định owncloud> \
--build-arg ADMIN_PASS=<password admin mặc định owncloud> \
-t mobidrive-image .

docker run -d -p 80:80 --name mobidrive mobidrive-image:latest

        