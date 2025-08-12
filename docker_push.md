A **Docker image can have multiple tags**. You can tag the same image as both `1.0` and `latest`, and later update `latest` when you build `1.1`.

## **1️⃣ Tag the Same Image with `1.0` and `latest`**
```sh
docker build -t my-app:1.0 -t my-app:latest .
```
This assigns **two tags** (`1.0` and `latest`) to the same image.

## **2️⃣ Push Both Tags to the Registry**
```sh
docker tag my-app:1.0 your-dockerhub-username/my-app:1.0
docker tag my-app:1.0 your-dockerhub-username/my-app:latest

docker push your-dockerhub-username/my-app:1.0
docker push your-dockerhub-username/my-app:latest
```
- Now `latest` and `1.0` both point to the **same image**.

## **3️⃣ Build `1.1` and Update `latest`**
Later, when you build a new version `1.1`, you can **reassign `latest` to the new build**:
```sh
docker build -t my-app:1.1 -t my-app:latest .
```
Then push:
```sh
docker tag my-app:1.1 your-dockerhub-username/my-app:1.1
docker tag my-app:1.1 your-dockerhub-username/my-app:latest

docker push your-dockerhub-username/my-app:1.1
docker push your-dockerhub-username/my-app:latest
```
Now `latest` points to `1.1`, but `1.0` is still available.

## **4️⃣ Verify the Tags**
List all local images:
```sh
docker images
```
You'll see something like:
```
REPOSITORY            TAG       IMAGE ID      CREATED        SIZE
your-dockerhub-username/my-app   latest    abc123456789  5 minutes ago   100MB
your-dockerhub-username/my-app   1.1       abc123456789  5 minutes ago   100MB
your-dockerhub-username/my-app   1.0       xyz987654321  10 days ago     98MB
```
🔹 `latest` and `1.1` **point to the same image (`abc123456789`)**  
🔹 `1.0` **remains unchanged (`xyz987654321`)**  

## **5️⃣ Pull & Run the Latest Version**
On another machine, you can pull and run the latest version:  
```sh
docker pull your-dockerhub-username/my-app:latest
docker run -d -p 5500:5500 your-dockerhub-username/my-app:latest
```
or a specific version:  
```sh
docker pull your-dockerhub-username/my-app:1.1
docker run -d -p 5500:5500 your-dockerhub-username/my-app:1.1
```

✅ **Now you can maintain multiple versions while always keeping `latest` updated!**

---
**RUN CONTAINER:**
```bash
docker run --rm  --name grank -p 5500:5500 --env-file .env -d grank
```