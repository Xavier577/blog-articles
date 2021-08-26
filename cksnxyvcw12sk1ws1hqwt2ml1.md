## Setting up a vanilla Typescript project the right way

Are you new learning typescript or you are an experienced typescript developer? chances are you have the urge to use it in any and every project rather than using plain javascript maybe because it gives you that sense of security. If by any chance you haven't  heard about typescript or have heard of it but never tried it (which i doubt is possible lol) ,Typescript is superset of Javascript which means it is essentially javascript with more features. Typescript is optionally statically typed which means you have to option to write code which has type checking during compile time. I am not gonna talk to much about that here, this is [eslam's article on typescript](https://eslam-7ot.hashnode.dev/typescript-basics-types) and also the offical [doc](https://www.typescriptlang.org/) if you want to learn more about it.


# A problem to be solved
 Typescript is awesome no doubt but sometimes using it with just plain html and css was hassle for me . Running the `tsc` command every five minutes to see my changes was really annoying . I even tried `tsc --watch`, this worked for a while but presented me with a problem, it would compile every  typecript file i create into their  separate javascript counterpart. And using custom modules became a problem. I began to wonder if there was a way to compile all my typescript code including modules into a single javascript bundle. I made my research and I discover there was a way.

#  The solution
After searching through google and eventually youtube, I realize that there weren't many articles or video to solve this problem, could it be that most people only use typescript in frameworks such as React or Angular and never use it with plain html or css? who knows. I eventually found a way thanks to [codingEnterpreneurs](https://www.youtube.com/c/CodingEntrepreneurs) youtube channel. He did a variation of this in his youtube channel on his learn typescript playlist you can checkout his playlist [here](https://youtu.be/yRQlo6ApYLw?list=PLEsfXFp6DpzQMickZgPq0Pn2uQD77UXoi). Subscribe to his channel if you find his content interesting (I am not an affiliate or anything I just value good content, although I wish I were I wouldn't be broke all the time lol). With that said , this was one of the few solutions I found which is what inspired me to write this article. Without wasting any more time let's get to it!

#  Getting started with the project

Before we anything we need the following for this to work:

<ul>
<li>nodejs</li>
<li>npm</li>
<li>git</li>
<li>typescript</li>
<li>webpack and webpack-cli</li>
<li>ts-loader</li>
</ul>

Now let's Go!

In your projects root directory:

```
npm init -y

```

> This will create a package.json file.

![packagejson-created.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629664613455/8--rNO4jD.png)


initialize git with:

```

git init

```

create .gitignore file (so we can ignore the node modules folder that will be created when we install our dependencies)

![gitignore-file-created.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629666629681/ndIvL0evG.png)

in the .gitignore file add:

```
node_modules

```

<h2>Installing the required dependencies</h2>
<hr/>

Install webpack and webpack-cli

```
npm install webpack webpack-cli

```

![installing_webpack.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629666677965/9PpvNnEQS.png)

Install typescript and ts-loader

```
npm install typescript ts-loader

```


![installing_typescript.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629666702101/z8piEaP_-.png)


You should now see all our installed dependencies in the package.json file under the dependencies property


![dependencies.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629666716303/JNUZ01tlZ.png)

<p>

Now we will create two folders:

<ul>
<li> public </li>
<li> src </li>
</ul>


![create_public_and_src_folders.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629666735322/K0HrpGa_O.png)
</p>

<p>

In public, we will create our index.html, static folder and our assets folder


![files_in_public.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629666810439/ncWPEVY5D.png)


index.html is our html file (obviously lol)

assets folder will contain files such as images,svgs, videos and so on

static folder will contain two additional folders:

<ul>
<li> css => this would contain our styles </li>
<li> bundle => this would be the destination of our compiled typescript code by webpack  </li>
</ul>

![files_in_static.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629666833927/U25DMCIpvr.png)

</p>

<p>

In src, create index.ts files and tsconfig.json


![files_in_src.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629666850732/Nkv7L1Ldy.png)

</p>

Add this to tsconfig.json:

```
{
  "compilerOptions": {
    "target": "es5",
    "module": "ESNEXT",
    "rootDir": "./",
    "strict": true,
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}

```

In the root directory, create webpack.config.js files and add this to the file:

```
const path = require("path");

module.exports = {
  mode: "production",
  entry: path.resolve(__dirname, "./src/index.ts"),
  module: {
    rules: [
      {
        test: /\.ts?$/,
        use: "ts-loader",
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: [ ".ts", ".js"],
  },
  output: {
    filename: "script.js",
    path: path.resolve(__dirname, "public", "static", "bundle"),
  },
};

```

<p>
what does this file do? well it essential gives webpack information such as what file to compile and what folder to compile to. For instance, the entry property above tells webpack what file we are compiling which is index.ts in src which we created earlier (note we used the path module to <em>resolve</em> the path to the index.ts file no pun intended). filename in the output property tells webpack what to name the compiled javascript bundle and path tells webpack where to place the javascript bundle which we have set to ./public/static/bundle (again we used the path module to resolve the path).You can change the name of the bundles output file to whatever you want just rember to point your html to that javascript bundle in your script tag.

</p>

<p>
open up the package.json file and under scripts


![scripts_packagejson1.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629666877517/LHOJJl5vF.png)

Add the start and build script shown below


![scripts_packagejson2.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629666890557/--39YbnNb.png)
</p>

> that wraps up everything with the configs

<b>We are almost there </b>

<p>
In our html let up set up our boilerplate and include our css and our bundled javascript (script.js which we named it in the webpack config this is important!!!)


![boilerplatehtml.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629666908124/tdt6h_2b8.png)

</p>

<p>
now in css (in the static folder) create style.css so we can add some styles <br/>


![styles.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629668955527/NyL-G9PEz.png)

</p>

<p>
<h3> Running your html</h3>
if you are like me and you are using vscode there is a 95% chance you have liveserver extension installed
run the html with liveserver or if you don't just open it in your browser. You should see this page: <br/>


![htmlPage.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629667025289/6nS73F4OI.png)

</p>

Open your terminal/command prompt

```
(assuming you are in the projects the root directory)

npm start

```

<p>

This should run the start script that we set to webpack --watch earlier. This compiles our index.ts and recompiles automatically on every change to index.ts . If everything was configured properly this should show up in your terminal. <br/>


![start_script_output.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629667064660/YETssKRSr.png)

and our script.js file should be generated in bundle <br/>


![bundle_created.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629667078483/gvbHfiXHz.png)

</p>

<p>
now that we are done with all the boring stuff let's write some typescript

in index.ts


![test1.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629667094799/1qy9MvSgC.png)

webpack automatically recompiles our code and the change should take effect in the browser because of liveserver.
we should see this: <br/>

![test1_result.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629667106358/rumoBgbwI.png)
</p>

<p>

now let's code up the logic for the counter example.

<ul>
<li>First we need to select the required html elements
 <ul>
<li>
our span tag with id counter-value
</li>
<li> 
our button with id increment-btn
</li>
<li>
our button with id decrement-btn
</li>
<ul>


![elements_to_select.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629667588641/Rs85orZT4.png)

![select_elements.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629667495057/e5vKbIf_T.png)
 </li>
<li> Next we create a new file in src called helpers which will contain two files
 <ul>
<li>
increment.ts which will contain our increment function which we will export

![incrementFunction.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629667758358/rNt3dCJba.png)
</li>
<li> 
 decrement.ts which will contain our decrement function which we will also export

![decrementFunction.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629667785122/tMwd6VH1q.png)
</li>
</ul>
 </li>
<li>
then will import these two functions in index.ts

![import_modules.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629668147650/QiyUxrTX3.png)
</li>
<li>
Finally the complete snippet


![complete_code_snippet.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629668297678/B9zSpl3pN.png)
</li>
<ul>

</p>

<h2>
 Increment count
</h2>
<hr/>


![count1.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629669482602/GZQaDLuH_.png)



![count2.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629669492238/caKa8lEqk.png)


![count3.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629669347988/OQet9xTcT.png)

<h2> Decrement count </h2>
<hr/>


![uncount1.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629669355006/yNZrUPWLD.png)


![uncount2.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629669363517/EGaQFqWXV.png)


![uncount3.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629669372391/ALCVgcWXE.png)

<h2>
Summary
</h2>
<p>
Now that we have everything is working fine it might appear like magic or maybe not but either way it is not magic. What essentially is happening is like I said before webpack would bundle all our typescript code including including all modules into a single file which we named script.js. The compiled code looks like this: <br/>


![compiled_output.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1629671587627/wDztqY4XS.png)

looks kinda ugly right? well this what is being run in the browser and we make changes in our typescript code webpack will recompile into the file which will trigger liveserver to reload, showing how changes automatically in the browser. 

This is a very simple example so you might node see the benefits straight away but in bigger projects relative to this were you don't want to use a framework this in my oppinon can be really helpful.
</p>

# Conclusion

Setting this up might seen like a hassle but using typescript with plain html and css has a lot advantages than with plain Javascript for example type checking, and also enables us to split out code into modules which makes everything much cleaner and easier to debug and much more. If you don't want to bother yourself with setting this up yourself you can make your own vanilla typescript project from this [boilerplate](https://github.com/Xavier577/vanilla-typescript-project-template) I set up so you don't have to set it up yourself. 

# A little bit about me the Author
If you have read this article to the end, I want you to know that i really appreciate it. My name is Joseph Tsegen, I have training to become a programmer now for 7 months now learning about the web and other technologies. I am aspiring to become a fullstack developer on the path to learn the GNERPT (Graphql Node Express React Postgresql Typescript), I also have interest in Python, Django and Kotlin. I am new to the community here on hashnode, this is in fact my first article please give it a like if you found it interesting or useful and comment on what you think about it and also what you think about building projects with this approach without any framework and what kind of projects you use this for. Feedbacks will be very helpful as it will help access myself and make better content, all feedbacks would be very appreciated. Let me know what you think I could have done better, thank you for reading.

