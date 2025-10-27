
1. login
```
docker login gitlab.local:5050
```

2. tag
```
docker tag milad-app:v1.0.0 gitlab.local:5050/iamsre/simple-node-app:v1.0.0
```

3. push
```
docker push gitlab.local:5050/iamsre/simple-node-app:v1.0.0
```

non-root user must have access token (PAT) with read and write registry permissions and be a mmember of project.
