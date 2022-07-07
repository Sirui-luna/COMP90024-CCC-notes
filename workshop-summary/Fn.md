# Build and deployment of Fn

These are the steps to follow in order to deploy an Fn instance running on a single computer.

The Fn project is an open-source container-native serverless platform that you can run anywhere -- any cloud or on-premise. It’s easy to use, supports every programming language, and is extensible and performant. (can create, aggregate fucntions and etc.)

## Install and start Fn

```shell script
(
  cd fn
  curl -LSs https://raw.githubusercontent.com/fnproject/cli/master/install | sh
  fn start --log-level DEBUG
)
```

(For instructions on operating systems othe than Linux, see [Quickstart](https://github.com/fnproject/fn#quickstart).
Also, you may need to prefix all the the `docker` and `fn` commands with `sudo`, depending on your Docker installation type.)


Check the Fn server has started by opening another shell:

```
docker ps
```


### Development of a Sample Function (Node,js)

**Create a local function directory**
> --runtime: what language used
> --trigger: communication protocol
> wcmp: the new Fn name

```
(  
  cd fn
  fn init --runtime node --trigger http wcmp 
)
```

A ==new directory== name `wcmp` should have been added; a few editings has to be done to its contents:

Suppose ==we want to fine-tune the Docker image that Fn builds by default==, we then have to add a Dockerfile and change the function runtime. 

**Fine tune the docker image**

Create a `Dockerfile` file under the `fn/wcmp` directory, with the contents:

```
FROM fnproject/node:dev as build-stage
WORKDIR /function
ADD package.json /function/
RUN npm install

FROM fnproject/node
WORKDIR /function
ADD . /function/
COPY --from=build-stage /function/node_modules/ /function/node_modules/
ENTRYPOINT ["node", "func.js"]
```

In the `func.yaml` file, change `runtime: node` to `runtime: docker` and delete the unnecessary `build_image`, `run_image`, and `entrypoint` entries.

**Define our own functions**

To have our ==functions== counts the input words, we have to replace `func.js` with:

```
const fdk = require ('@fnproject/fdk');

fdk.handle ( (input) => {
  let counts = {};
  input.toLowerCase ()
    .split(/\W+/)
    .filter ((w) => {
      return w.length > 1;
    })
    .forEach ((w) => {
      counts[w] = (counts[w] ? counts[w] + 1 : 1);
    });
  return counts;
})
```

**Create an Fn app**

(these commands have to be run from the directory containing the `wmcp` function directory):

```
(
  cd fn
  fn create app wcapp
)
```
**Function build and deploy**

Function build and deployment on the local Fn server:

```
(
  cd fn
  fn deploy --app wcapp --verbose --local wcmp
  fn list functions wcapp 
)
```

**Function invokation**:（in docker）
> --data: followed by input

```
curl -XPOST 'http://localhost:8080/t/wcapp/wcmp' --data 'lorem ipsum'
```


## Start the Fn Web-admin

**Start the Fn webadmin user interface:**

```
docker run --rm -it --link fnserver:api -p 4000:4000 -e FN_API_URL=http://api:8080\
  -e FN_LISTENER=unix:/iofs/lsnr.sock fnproject/ui
```
can open the dashboard:
Point your browser to `http://0.0.0.0:4000`. 


## Testing Fn scalability

In another shell, download Wikipedia pages for the test

Type this in terminal:

```
pages="italy china france germany australia brazil denmark mexico zambia thailand"
for page in ${pages};
   do curl "https://en.wikipedia.org/w/api.php?format=xml&action=parse&prop=wikitext&page=${page}"\
      -o /tmp/${page}.xml      
done
```

Call the word-count functions on the downloaded pages concurently whilelooking at the `wcapp` app statistics on the UI:

Type this in terminal:

```shell script
for i in {1..100};
  do
    for page in ${pages};
      do curl -XPOST 'http://localhost:8080/t/wcapp/wcmp' --data @/tmp/${page}.xml &      
    done
  done
```

On the UI window, ==the number of replicas(i.e.functions)== can be observed to raise, and then fall.(where to determine the number of replica we needed?)


### Development of a Sample Function (Python)

Sample function creation, can specify the runtime to be any languages

```shell script
(
  cd fn
  fn init --runtime python --trigger http pywcmp 
)
```

A new directory name `pywcmp` should have been added; a few editings has to be done to its contensts:

Replace `func.py` with:

```
import io
import json
import logging

from fdk import response

def handler(ctx, data: io.BytesIO = None):
    try:
        counts = dict()
        logger = logging.getLogger()

        for word in data.getvalue().split():
            counts[word.decode('utf-8')]= (counts.get(word.decode('utf-8')) or 0) + 1

    except (Exception, ValueError) as ex:
        logging.getLogger().info('error: ' + str(ex))

    return response.Response(
        ctx, response_data=json.dumps(counts), headers={"Content-Type": "application/json"}
    )
```

Function build and deployment on the lcoal Fn server (these commands have to be run from the directory containing the `wmcp` funciton directory):

```shell script
(
  cd fn
  fn deploy --app wcapp --local pywcmp
  fn list functions wcapp 
)
```

Function invokation:

```shell script
curl -XPOST 'http://localhost:8080/t/wcapp/pywcmp' --data 'lorem ipsum'
```
