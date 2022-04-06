## How to speed up your server response time with caching

Caching is a well known technique used to speed up server response time and provide better user experience. Today we would be diving into, what this technique is all about, how it works and how we can use this technique in our own application.

###  What is caching 
Caching is the technique of storing a copy of data in memory and then serving it when needed.
So, instead of re-fetching data from a database or a third-party API, the copy of the data in the cache (saved in memory) would be returned, and because data kept in memory can be accessed faster than data stored on disk, the process would be very fast. 

### How Caching works
We should have a high level understanding of how caching works after learning what it is.
So now we'll talk about how it works from an implementation standpoint; trust me, it's actually pretty straightforward. So the logic goes like this: we have a user who requests data from the server; we check if the data is available in the cache(in our case a store like Redis); if it is, we return it to the user; if it isn't, we query the data from our database or a third-party API and save it to the cache; then the next time the user requests the same data, we retrieve it from the cache and return it to the user. 

![Did-you-know_-Redis-Fact-1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649186989190/boVfEINGA.png)

### What we would be working on
we would be looking at a pre-existing server that I have built for this guide and we are gonna use caching to improve it's response time. if you want to follow along you can use exode-cli (I created exode btw you can check it out on npm [here](https://www.npmjs.com/package/exode)) to create a project with `npx exode init projectName` or `npx exode init projectName -js` (to use Javascript because exode uses typescript by default to justify my role as a Typescript evangelist 😂) or use any other method of your choosing.

### Server response time without caching
 > The main code of our App

```
import express, { Request, Response, NextFunction as Next } from "express";
import axios from "axios";
import cors from "cors"
import morgan from "morgan";
import { PromiseWrapper } from "./helpers/async-handler";

const app = express();

app.use(cors({ origin: "*"})) // don't do this in your production apps, always restrict your cors origin
app.use(express.json())
app.use(express.urlencoded({extended: false}))
app.use(morgan("dev"))

app.get("/", (_req, res) => {
  res.send("<h1>let's code!</h1>");
});

app.get("/api/photos", async (_req: Request, res: Response, next: Next) => {
  const url = "https://jsonplaceholder.typicode.com/photos";
  const {result: response, error: fetchError} = await PromiseWrapper(axios.get(url)); // fetches 5000 images

  if (fetchError) return next(fetchError);
  return res.json(response?.data);
});

export default app;
```

> Breakdown of what the server does

It's a simple server with two endpoint first the default `GET /` route and the `GET /api/photos`.

- `GET /` -> Outputs "Let's code" as the response
- `GET /api/photos` -> fetches 5000 pictures from the [jsonplaceholder](https://jsonplaceholder.typicode.com) API

> if you are wondering what PromiseWrapper does

```
export const AsyncWrapper = async <T>(asynFn: () => Promise<T>) => {
  try {
    let result = await asynFn();
    return { result };
  } catch (err) {
    let error = err;
    return { error };
  }
};

export const PromiseWrapper = async <T>(promise: Promise<T>) => {
  try {
    let result = await promise;
    return { result };
  } catch (err) {
    let error = err;
    return { error };
  }
};

```

> Response time (using postman as our API client)

Fetching those 5000 photo data from the jsonplaceholder api is gonna take quite some time. And doing that for every single request is not gonna give a good user experience. using postman as our API client we can see it takes quite some time.


![Screenshot at 2022-04-05 21-45-32.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649192214837/jquTSakRI.png)


![Screenshot at 2022-04-05 21-58-48.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649195903732/g2GyVozTQ.png)


![Screenshot at 2022-04-05 21-59-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649195956299/3GIeV00fg.png)


![last_without_cashing.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649196080649/6MC1dR9c6.png)


Of the 4 tests we did,the response times were  `9.89s`, `7.78s`, `3.67s`, and `2.67s` respectively. on the surface that doesn't look like much but for our users that is quite some time which would negatively impact their user experience.


### Server response time after implementing caching

Now it's time for use to implement caching to this application.We would be using redis as our store, so let's install ioredis to use as our redis driver (you need to have redis installed).

```
$ yarn add ioredis
```

> To make sure Redis is running in the background 

```
# on Mac and Linux
$ redis-server --daemonize yes 
```

> Let's modify our code to use caching

```
import express, { Request, Response, NextFunction as Next } from "express";
import axios from "axios";
import cors from "cors";
import morgan from "morgan";
import Redis from "ioredis";
import { PromiseWrapper, AsyncWrapper } from "./helpers/async-handler";

const app = express();

const RedisClient = new Redis();

RedisClient.on("connect", () => console.log("redis connected!"));
RedisClient.on("error", (err) => {
  console.error(err);
  RedisClient.quit();
});

app.use(cors({ origin: "*" })); // don't do this in your production apps, always restrict your cors origin
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(morgan("dev"));

app.get("/", (_req, res) => {
  res.send("<h1>let's code!</h1>");
});

app.get("/api/photos", async (_req: Request, res: Response, next: Next) => {
  const retrievePhotosFromCache = async () => RedisClient.get("photos")
  const { result: cachedPhotos, error: cacheRetrieveError } = await AsyncWrapper(retrievePhotosFromCache);
  
  if (cacheRetrieveError) console.error(cacheRetrieveError);
  if (cachedPhotos) return res.json(JSON.parse(cachedPhotos));

  const url = "https://jsonplaceholder.typicode.com/photos";
  const { result: response, error: fetchError } = await PromiseWrapper(axios.get(url));
  const data = response?.data;
  const dataToCache = JSON.stringify(data)

  if (fetchError) return next(fetchError);

  const savePhotosToCache = async () => RedisClient.set("photos", dataToCache)
  const { error: cacheSaveErr } = await AsyncWrapper(savePhotosToCache);

  if(cacheSaveErr) console.error(cacheSaveErr)
  return res.json(data);
});

export default app;
```

We import our Redis driver, we initialize a new instance and listen for two events; `connect` and `error`. when we connect we log out to indicate that redis has been connected to our app and if there is an error, log out the error and disconnect from redis to prevent indefinite retries.

The logic of our implementation works exactly as we have talked about earlier. so let's run our server and see the how this affects our response time.
 
```
$ yarn dev
```

```
# expected output
$ nodemon --exec ts-node ./src/index.ts --mode development
[nodemon] 2.0.15
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: ts,json
[nodemon] starting `ts-node ./src/index.ts --mode development`
listening on http://localhost:8080
redis connected!
```
 
> Response time (After the initial request)

![Screenshot at 2022-04-05 22-41-24.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649196428370/USigkoO2z.png)


![Screenshot at 2022-04-05 22-49-07.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649196431074/h-A44DSHx.png)

![Screenshot at 2022-04-05 22-49-16.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649196433945/WwpMxq84w.png)

![Screenshot at 2022-04-05 22-49-25.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649196436524/qxNMoM6eZ.png)

Of the 4 tests we did,the response times were  `198ms`, `117ms`, `78ms`, and `84ms` respectively (note that 1000ms is equivalent to 1s). Which is a significant improvement in the response time from our previous implementation without caching, roughly 50 times faster (comparing the average of both implementations).


### conclusion

We have seen that the response time of our server can be significantly improved with caching (check out the codename on [github](https://github.com/Xavier577/reponse_time_improvement_with_caching)). The are other usecases for caching such as caching request on a NGNIX server, caching static assets on a CDN and so on. Caching works well to reduce latency in web servers. Other server optimization techniques such as load balancing, using clusters/child processes and so on. This was just a basic usecase but we learnt from it, we'll meet again in the next one.




