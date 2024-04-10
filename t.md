This is API + JSON: upload file and save data to JSON file.

Cors: CORS is a security mechanism that controls access to resources from different origins in a web browser. It allows web applications running at one origin to request resources from a different origin, solving the problem of cross-origin requests.

```javascript
// .env: PORT = 5050
// Server.js
require("dotenv").config();
const express = require("express");  // import Express framework
const cors = require("cors");  // CORS:cross-origin resource sharing middleware. 
const app = express();
//这一行告诉 Express 应用在处理所有路由请求之前先使用 CORS 中间件。cors() 函数返回一个中间件，用于处理跨域请求，并允许来自所有源的请求访问资源
app.use(cors());

const getVideos = require("./routes/videos.js");
const PORT = 5050; 

//This is middleware that allows use to send JSON requests
app.use(express.json());
app.use("/videos", getVideos);

app.listen(PORT, ()=> {
    console.log(`${PORT} is listening...`);
})

// routes/videos.js
const express = require("express");
const router = express.Router();
const fs = require("fs");
// const { randomUUID } = require("crypto");

const { v4: uuidv4 } = require("uuid");

const FILE_PATH = "./data/videos.json";

const readData = () => {
  const videoData = fs.readFileSync(FILE_PATH);
  const parsedData = JSON.parse(videoData);
  console.log(parsedData);
  return parsedData;
};

router.get("/", (req, res)=> {
    const videos = readData();
    res.json(videos);
})

router.get("/:videoId", (req, res) => {
  const videos = readData();
  const selectedVideo = videos.find((video) => video.id == req.params.videoId);
  res.json(selectedVideo);
});

router.post("/", (req, res)=> {
    const newVideo = {
      id: uuidv4(),
      title: req.body.title,
      channel: "Todd Welch",
      image: "https://project-2-api.herokuapp.com/images/image1.jpg ",
      description: req.body.description,
      views: "0",
      likes: "0",
      duration: 4,
      timestamp: Date.now(),
      comments: []
    };
    const videos = readData();
    videos.push(newVideo);
    fs.writeFileSync("./data/videos.json", JSON.stringify(videos));
    res.status(201).json(newVideo);
})

module.exports = router;

```

In React, at the root of the project, create a `.env` file with the following content:

`REACT_APP_SERVER_URL=http://localhost:5050`

In the application, this value can simply be read as such:

`const {REACT_APP_SERVER_URL} = process.env` or `const REACT_APP_SERVER_URL = process.env.REACT_APP_SERVER_URL`

Assuming the application is created with `create-react-app`, add the `dotenv` package;

```react
// REACT_APP_SERVER_URL=http://localhost:5050
import Header from "./components/Header/Header";
import Home from "./pages/Home/Home";
import UploadPage from "./pages/UploadPage/UploadPage";
import { BrowserRouter, Routes, Route } from "react-router-dom";

function App() {
  return (
    <BrowserRouter>
      <div className="App">
        <Header />
        <main>
          <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/videos/:videoId" element={<Home />} />
            <Route path="/upload" element={<UploadPage />} />
          </Routes>
        </main>
      </div>
    </BrowserRouter>
  );
}

export default App;

```

```react
import React from "react";
import "./Home.scss";
import { useEffect, useState } from "react";
import { useParams } from "react-router-dom";
import Hero from "../../components/Hero/Hero";
import VideoList from "../../components/VideoList/VideoList";
import axios from "axios";
import CommentAll from "../../components/CommentAll/CommentAll";

const REACT_APP_SERVER_URL = process.env.REACT_APP_SERVER_URL;
console.log("REACT_SERVER_URL:", REACT_APP_SERVER_URL);

function Home() {
    const [videos, setVideos] = useState([]);
    const [selectedVideo, setSelectedVideo] = useState([]);
    const { videoId } = useParams();

    const getCertainVideo = async () => {
      try {
        const response = await axios.get(`${REACT_APP_SERVER_URL}/videos`);
        setVideos(response.data);
        if (videoId) {
            const responseVideo = await axios.get(`${REACT_APP_SERVER_URL}/videos/${videoId}`);
            setSelectedVideo(responseVideo.data);
        } else {
            setSelectedVideo(response.data[0]);
        }
      } catch (error) {
        console.log(error);
      }
    };

    useEffect(() => {
      getCertainVideo();
    }, [videoId]);

    const filteredVideos = videos.filter(
      (video) => video.id !== selectedVideo.id
    );
    return (
      <div className="App">
        <Hero selectedSrc={selectedVideo.image} />
        <section className="feature">
          <div className="feature__wrapper">
            <CommentAll selectedVideo={selectedVideo} />

            <section className="feature__right">
              <VideoList filteredVideos={filteredVideos} />
            </section>
          </div>
        </section>
      </div>
    );
}

export default Home;
```

```React
// CommentAll.js
import { AddCommentIcon } from "../Icons/Icons";
import views from "../../assets/images/Icons/views.svg";
import likes from "../../assets/images/Icons/likes.svg";
import HeaderImg from "../HeaderImg/HeaderImg";
import "./CommentAll.scss";
import { toDate } from "../../functions/CommonFunction";
import Comment from "../Comment/Comment";


function CommentAll({ selectedVideo }) {
  return (
    <div className="feature__left">
      <h1>{selectedVideo.title}</h1>
      <div className="feature__highlight">
        <div className="feature__author">
          <h5>{selectedVideo.channel}</h5>
          <p>{toDate(selectedVideo.timestamp)}</p>
        </div>
        <div className="feature__likes">
          <div className="feature__align">
            <img src={views} alt="Views" />
            <p>{selectedVideo.views}</p>
          </div>
          <div className="feature__align">
            <img src={likes} alt="Likes" />
            <p>{selectedVideo.likes}</p>
          </div>
        </div>
      </div>
      <p className="p-margin">{selectedVideo.description}</p>

      <h3>
        {selectedVideo.comments ? selectedVideo.comments.length : 0} Comments
      </h3>
      <div className="feature__totalComments">
        <HeaderImg />

        <div className="feature__subComments">
          <div className="feature__inputComment">
            <p htmlFor="name">JOIN THE CONVERSATION</p>
            <input type="text" name="name" id="name" className="text" />
          </div>
          <button>
            <AddCommentIcon /> <span className="__btnText">COMMENT</span>
          </button>
        </div>
      </div>
      {selectedVideo.comments ? selectedVideo.comments.map((comment) => (
        <Comment
          key={comment.id}
          id={comment.id}
          name={comment.name}
          day={toDate(comment.timestamp)}
          commentDetail={comment.comment}
        />
      )): <><p>No Comments</p></>}
    </div>
  );
}

export default CommentAll;
```

```React
// VideoList.js
import { Link, useNavigate } from "react-router-dom";
import VideoDetails from "../VideoDetails/VideoDetails";

export default function VideoList({ filteredVideos }) {
  const navigate = useNavigate();
  const handleSubmit = (event) => {
    event.preventDefault();
    alert("Your upload was successful!");
    navigate("/");
  };
  return (
    <>
      <h2>NEXT VIDEOS</h2>
      <form onSubmit={handleSubmit}>
        {filteredVideos.map((video) => (
          <article className="click" key={video.id} >
              <Link to={`/videos/${video.id}`}>
                <VideoDetails key={video.id} video={video} />
              </Link>
          </article>
        ))}
      </form>
    </>
  );
}
```

```React
// UploadPage.js
import "./UploadPage.scss";
import video_img from "../../assets/images/Upload-video-preview.jpg";
import { UploadIcon } from "../../components/Icons/Icons";
import {useState} from "react";
import axios from "axios";
import { useNavigate } from "react-router-dom";
const { v4: uuidv4 } = require("uuid");
const REACT_APP_SERVER_URL = process.env.REACT_APP_SERVER_URL;

export default function UploadPage() {
  const [name, setName] = useState("");
  const [comment, setComment] = useState("");
  const navigate = useNavigate();
  const changeName = (e)=> {
    setName(e.target.value);
  }
  const changeComment = (e) => {
    setComment(e.target.value);
  }
  const addVideo = async (e)=> {
    e.preventDefault();
    const title = e.target.title.value;
    const description = e.target.description.value;

    try {
      await axios.post(`${REACT_APP_SERVER_URL}/videos`, {
        id: uuidv4(),
        title: title,
        description: description,
      });
      e.target.reset();
      alert("Your upload was successful!");
      navigate("/");
    } catch (error) {
      console.log(error);
    }
  }
  
  return (
    <div className="load">
      <form onSubmit={addVideo}>
        <h1>Upload Video</h1>
        <div className="load__wrapper">
          <div className="load__title">
            <label htmlFor="image">VIDEO THUMBNAIL</label>
            <img
              className="load__image"
              src={video_img}
              alt="THUMBNAIL"
            ></img>
          </div>
          <div className="load__details">
            <label htmlFor="title">TITLE YOUR VIDEO</label>
            <input
              type="text"
              name="title"
              id="title"
              placeholder="Add a title to your video" onChange={changeName}
            ></input>

            <label htmlFor="description">ADD A VIDEO DESCRIPTION</label>
            <input
              type="text"
              name="description"
              id="description"
              placeholder="Add a description to your video" onChange={changeComment}
            ></input>
          </div>
        </div>
        <div className="load__right">
          <button className="load__btn">
            <UploadIcon />
            <span className="load__btnText">PUBLISH</span>
          </button>
          <a className="load__cancleText" href="">
            CANCEL
          </a>
        </div>
      </form>
    </div>
  );
}
```

1. **What is middleware in the context of web development?** Middleware is software that sits between an application's core logic and the underlying operating system or framework. It intercepts requests and responses, allowing developers to add additional functionality such as logging, authentication, and error handling. (拦截请求和响应)
2. **How does middleware work in Express.js?** In Express.js, middleware are functions that have access to the request object (`req`), response object (`res`), and the `next` function in the application's request-response cycle. Middleware functions can perform tasks such as modifying request or response objects, terminating the request-response cycle, or passing control to the next middleware function by calling the `next()` function.
3. **What are some common use cases for middleware?** Common use cases for middleware include authentication (verifying user credentials before accessing protected routes), authorization (checking user permissions), logging (recording request and response data for debugging and auditing purposes), error handling (catching and handling errors in the application), and request processing (parsing request bodies, handling file uploads).
4. **How do you create custom middleware in Express.js?** Custom middleware in Express.js can be created by defining a function that takes three arguments: `req` (the request object), `res` (the response object), and `next` (a callback function to pass control to the next middleware in the chain). This function can perform any desired operations and then call `next()` to pass control to the next middleware. Custom middleware can be used globally for all routes or applied to specific routes as needed.
5. **What is the difference between application-level middleware and route-level middleware in Express.js?** Application-level middleware in Express.js is applied to every request that the application receives. It can be set up using `app.use()` and affects all routes defined in the application. Route-level middleware, on the other hand, is applied only to specific routes or groups of routes. It can be set up using `app.use()` with a specific route path or by using `router.use()` within route-specific middleware. Route-level middleware is useful for handling request-specific tasks or for organizing middleware based on route structure.

1. **What is Axios and why is it used?**

   - Axios is a JavaScript library used for making HTTP requests from web browsers and Node.js. It simplifies the process of making asynchronous HTTP requests and provides a more intuitive API compared to native browser APIs like `fetch`. Axios is used to interact with web servers, consume APIs, and fetch data from external sources in web applications.

2. **How do you make a GET request using Axios?**

   - To make a GET request using Axios, you can use the `axios.get()` method and provide the URL of the resource you want to retrieve. For example:

     ```javascript
     axios.get('https://api.example.com/data')
       .then(response => {
         console.log(response.data);
       })
       .catch(error => {
         console.error('Error fetching data:', error);
       });
     ```

3. **What are the advantages of using Axios over native fetch API?**

   - Axios offers several advantages over the native fetch API, including:
     - Support for older browsers.
     - Built-in support for handling JSON data.
     - Simpler syntax for making requests.
     - Features such as request and response interception, automatic request cancellation, and more.

4. **How do you handle errors in Axios requests?**

   - Axios provides built-in error handling by automatically rejecting the promise if a request fails. You can use the `catch()` method to handle errors and perform appropriate actions. For example:

     ```javascript
     axios.get('https://api.example.com/data')
       .then(response => {
         console.log(response.data);
       })
       .catch(error => {
         console.error('Error fetching data:', error);
       });
     ```

5. **Explain Axios interceptors and their use cases.**

   - Axios interceptors allow you to intercept and modify HTTP requests and responses before they are sent or processed. They are useful for implementing features such as adding authentication headers, logging request and response data, implementing request retry logic, and handling global error handling. Interceptors can be defined using `axios.interceptors.request.use()` for request interceptors and `axios.interceptors.response.use()` for response interceptors.