---
layout: post
title: SPA with FastAPI
category:
tags:
tagline:
---

Being on the hunt for better ways to prototype applications led me to try learning `vue.js` with Python's `FastAPI`. It turns out this combination is surprisingly potent!

```html
<html>
  <title></title>
  <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
  <script src="https://unpkg.com/axios/dist/axios.min.js"></script>

  <body>
    <div id="app">
      <textarea
        name=""
        id="content"
        cols="30"
        rows="10"
        v-on:keyup="myapi"
        v-model="content"
      ></textarea>
      <p id="output" v-model="output"></p>
    </div>

    <script>
      new Vue({
        el: "#app",
        data: {
          title: "",
          content: "",
        },
        methods: {
          myapi() {
            return axios
              .post(
                "/myapi",
                {
                  content: this.content,
                },
                {
                  headers: {
                    "Content-type": "application/json",
                  },
                }
              )
              .then((response) => {
                console.log(response.data);
                document.getElementById("output").textContent = response.data;
              });
          },
        },
      });
    </script>
  </body>
</html>
```

```py
from fastapi import FastAPI, Request
from fastapi.templating import Jinja2Templates
from pydantic import BaseModel

templates = Jinja2Templates(directory="templates")

app = FastAPI()


class TextArea(BaseModel):
    content: str


@app.post("/myapi")
async def post_textarea(data: TextArea):
    return f"length of input is: {len(data.content)}"


@app.get("/")
async def serve_home(request: Request):
    return templates.TemplateResponse("home.html", {"request": request})
```

Happy coding!

Based on this: https://stackoverflow.com/questions/64522736/how-to-connect-vue-js-as-frontend-and-fastapi-as-backend
