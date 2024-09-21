```bash
sudo nano ~/.docker/config.json
```

```txt
{
        "auths": {
                "https://index.docker.io/v1/": {
                        "auth": "c3luYXB0YWh1YjpBTjNGNmZLTUJGWkRCcFNYNnZQMw=="
                }
        },
        "credsStore": "desktop", # <---- Remove this in order to allow bash login
        "currentContext": "desktop-linux",
        "plugins": {
                "debug": {
                        "hooks": "exec"
                },
                "scout": {
                        "hooks": "pull,buildx build"
                }
        },
        "features": {
                "hooks": "true"
        }
}
```
