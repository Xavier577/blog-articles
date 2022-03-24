## Setting up a nodejs project with typescript and deploying to heroku

Setting up nodeJs with typescript can be quite troublesome if not done properly. To make this worse you probably won't see the issue with the way you set your project up until you want to deploy it. Today we are gonna go over how exactly we can set up nodeJs with typescript in a way that would make deployment easy.

# Getting started

For this tutorial we are gonna be using the following packages: 
<ul>
<li> yarn </li>
<li> nodejs </li>
<li> typescript </li>
<li> ts-node </li>
<li> express </li>
<li> nodemon  </li>
<li> @types/node (type definitions for nodejs) </li>
<li> @types/express (type definition for typescript) </li>
</ul>

# Initializing the project

we are going to initialize the project with yarn and git

```
$ yarn init -y

```

```
$ git init

```

we should have our package.json file create for us like so

![package_json_created.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632520932783/2sCNW9nK1.png)

# Installing the dependencies for the project

To make our lives easier let's install express

```
$ yarn add express

```

> Installing typescript and ts-node as dev dependencies

```
$ yarn add -D typescript ts-node

```

> installing the type definition for node and express

```
$ yarn add -D @types/node @types/express

```

> and finally nodemon to run our server in development

```

$ yarn add -D nodemon

```

After installing our dependencies it should be included in the package.json file

![package_json_postinstall.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632521040738/8iLV5N_qf.png)

And our node_modules folder must have been created

![folder_structure_step2.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632521068567/exkR_qz4m.png)

Let's create a .gitignore file and add our node_modules in there to prevent it from being tracked by git

```
// (in .gitignore)

node_modules

```

# Adding the scripts

> add the following to your package.json file

```
  "scripts": {
    "start": "node dist/index.js",
    "build": "tsc",
    "dev": "nodemon --exec ts-node ./src/index.ts --mode development"
  }

```

our package.json should look like this

![package_json_postscript.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632521099391/5ddsWf2Tp.png)

<h2>Explaination for the script</h2>
<ul>
<li> The build script is responsible for  making our javascript build of the project for deployment (it would build all our files into a dist folder with the tsc command which we would set up in a second) </li>
<li>start is what would run the app when we deploy it to service such as heroku</li>
<li>dev is to run our app in development</li>
</ul>


<h1> Generating our tsconfig file </h1>



Normally we would run `tsc --init` to create our tsconfig.json file but in this tutorial i would use a library created by one of my favourite youtubers ([Ben Awad](https://www.youtube.com/c/BenAwad97)) called tsconfig.json.
To use the package we'll run:

```
$ npx tsconfig.json

```

![benawads_tsconfig.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632521146710/kN5vap8eS.png)

I'll select nodejs and that should create a tsconfig for us.

![tsconfig_benawad.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632521166007/dGNke16s8.png)


so it contains some set rule for our project, feel free to change whatever you like to your liking but the most important options here are "outDir" which tells the typescript compiler where to compile our javascript file, "excludes" which tells typescript what files to ignore, "include" tells typescript what files to check during compilation, "target" the version of javascript we are compiling to (which Ben has set to es2017) and "module" which tells typescript what module syntax should be used.
<br/>
The "outDir" has been set to dist but you can name it whatever you want (but remember to change that in the package.json file) but by convention it is usually best practise to call it dist or build.
<br/>

we are gonna want to include the dist folder to the .gitignore file as this folder is only needed during deployment .
 

<h2>Creating our source folder </h2>

First we are gonna want to create a folder called "src", this is where we would write all our typescript code

![final_folder_structure.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632521499317/5f4OwY_bd.png)

I'll create a file called index.ts which would be our main input

![indexts_stage0.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632521523230/e9T9i0VCv.png)

# Now let's write some code

![code_snippet_1.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632521679053/GesTeL_86.png)

Here we are importing express, setting the a new express application to the variable app.  We set the PORT variable to use the process environment port if available and 8080 otherwise then we listen then PORT variable. And we handle our get request with the app.get() method by sending to the client "hello" + client hostname(which is localhost in this case)
<br/>

<h2>now let's run our server</h2>
<br/>

![start_dev_server.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632522291464/jiXMzOlZC.png)
<br/>

<h2> now let's run it in the browser </h2>
<br/>

![browser_test_1.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632522322875/z0jZnutEr.png)

<h2>Let's make a change</h2>

![ts-error-1.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632522346876/mqHJ-m8HP.png)

<br/>

That threw an error because in the tsconfig.json the "noUnusedParameters" property is set to true which prevent the unused parameters in the code (which is some best practice stuff :(   ).

![ts_conflit1.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632522387777/58di_uSpW.png)

To remove the error you can either disable the rule by setting "noUnUsedParameters" to false in the tsconfig.json file or simply use the editor quick fix option which I am gonna use.

![quick_fix_1.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632522411694/WEHVSAmPz.png)
<br/>

![quick_transition_1.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632522439204/OaU2WNh7w.png)

<br/>

![fix1.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632522486329/mQja1s-F3.png)

That was fixed by placing an underscore on the unused parameter, now let's restart the server

![restart1.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632522500377/pF0MeLtTl.png)

![change1.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632522509672/FvhUmH1LD.png)

It works yay!

# Deploying our app to heroku

Now let's see how exactly we can deploy our app which is the part which cause alot of issues especially when unusing typescript with nodejs.

first i am gonna run the build command so that we can see how our app would be built.

we'll run:

```
$ yarn run build

```

This should create a folder dist which contains our compiled code

![build_files.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632522583571/z3faCKpOS.png)

This is gonna be what would be run on heroku when we deploy

I am gonna delete this file for now.

Assuming we did everything correctly up until this point, i am gonna list the step for deploying this add to heroku
via the heroku-cli. If you don't have installed you can install it with `npm install -g heroku`

<ul>
<li>

<h2>Logging in to heroku</h2>
<br/>

![heroku_login.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632522601501/gf1ZX6Q28.png)

</li>
<li>
<h2>Adding changes to git</h2>

```
$ git add .
```

</li>
<li>
 <h2>Committing changes</h2>

```
$ git commit -am "deploy"
```

</li>
<li>
  <h2>Adding heroku remote</h2>
<br/>

![heroku_remote.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632522623092/A5PohYpJg.png)

</li>
<li>
  <h2>Pushing to heroku</h2>

```
$ git push heroku master
```

![deploying1.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632522649820/btsEx5kyM.png)

![deploying_complete.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632522665183/c-3Ze3CeK.png)

</li>
<ul>

# Deployed site 🚀🚀🚀

![deployed_site.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1632522674943/DIqi7J385.png)

yay! we have our site deployed.

# Conclusion
This is just a very basic example but the main purpose of this guide was to help give an idea on how you can host/deploy a nodejs typescript project of any sort. You can change the rules in the tsconfig.json file to match your use case but make sure the "outDir" is correctly specified.
            I have had issues deploying typescript nodejs projects myself but now I feel a little more confident doing it now so I decided to share it with the community. I hope this was able to help out in someway. Lemme know if there are better ways to do this. Please do like and comment if you found this interesting.




