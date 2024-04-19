# Instructions on how to run a wandb server locally

## Prereqs

* wandb

```
(venv) $ pip install wandb
```

* Docker engine is used under the hood in the `wandb server start` command to manage various container images needed to run wandb. 
Further investigation is needed to see if wandb server start works with podman.

### Start wandb

```
wandb server start
```

![image info](./screenshots/screenshot-wandb-local-1.png)

### Go to http://localhost:8080 in your browswer

![image info](./screenshots/screenshot-wandb-local-2.png)

### Click "Login", create an account. 

Add your fullname, email, username, and a password.

Use a valid email and don't forget username and password.

### Click on "Get a free license"

![image info](./screenshots/screenshot-wandb-local-3.png)

### Click on "Get a free license"

![image info](./screenshots/screenshot-wandb-local-4.png)

### Click on "Continue"

![image info](./screenshots/screenshot-wandb-local-5.png)

### Click on "New Organization"

![image info](./screenshots/screenshot-wandb-local-6.png)

### Add an Organization Name then click "Next"

![image info](./screenshots/screenshot-wandb-local-7.png)

### Click "Generate License Key"

![image info](./screenshots/screenshot-wandb-local-8.png)

### Click "Copy License"

![image info](./screenshots/screenshot-wandb-local-9.png)

### Click "Copy"

![image info](./screenshots/screenshot-wandb-local-10.png)

### Go back to localhost:8080, click on "Add license"

![image info](./screenshots/screenshot-wandb-local-11.png)

### Paste license into text box

The page should go from this:

![image info](./screenshots/screenshot-wandb-local-12-1.png)

To this:

![image info](./screenshots/screenshot-wandb-local-12-2.png)

### Click "Update settings"

![image info](./screenshots/screenshot-wandb-local-13.png)

### Go back to localhost:8080, copy API key

![image info](./screenshots/screenshot-wandb-local-14.png)

### Use key in below python program called wb-run.py

```python
import wandb

WANDB_BASE_URL = "http://localhost:8080"
WANDB_API_KEY = <WANDB API KEY GOES HERE>
wandb.login(host=WANDB_BASE_URL, key=WANDB_API_KEY)
run = wandb.init(project="foo-project",group="foo-group",job_type='foo-job-type')
run.finish()
```

### Run the following python program

```sh
python wb-run.py
```

### See run under My projects

![image info](./screenshots/screenshot-wandb-local-15.png)


### The End. You now are ready to integrate wandb into your code and see what you've done locally!
