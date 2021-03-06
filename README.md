# [DynoSrc](http://www.dinosrc.it)

DynoSRC is a general solution for efficiently delivering JS resources to clients, using diff-based updates as assets change over time.

## Benefits

#### Minimize HTTP Requests
DynoSRC loads JavaScript files inline in your HTML response, then stores them in localStorage. You can even inline the calls to the DynoSRC client lib in your HTML response, eliminating all HTTP requests for JavaScript on your site.

#### Differential Updates
Normally, if a JS asset on your site changes, your users will have to download the entire file again even though just a fraction of it changed. DynoSRC sends down differentials updates so changes to large files don't require full downloads.

## Getting Started

#### Install DynoSrc

    npm install dynosrc

#### Configure Middleware

    dynSrc.globals({
      //folder containing your JS / CSS files
      assetsDir: __dirname + '/assets'
    });

    //install express style middleware
    app.use(dynoSrc.middleware());

    dynSrc.assets({
      //point directly to files on github
      'ryanstevens/ModelFlow': {
        filename: 'package.json',
        source: 'git',
        //this can also be a tag
        head: '8050f1'
      },
      'jquery': {
        head: '1.9.1',
        //this lives on disk
        source: 'asset'
      }
    });

#### Get Patches, Send to Client

    app.get('/', function (req, res) {
      dynSrc.getPatches(req, {
        patches: ['jquery', 'backbone']
      }, function(err, patches) {
        res.render('index.html', {
          title: 'Super Cool Project',
          patches: patches
        });
      });
    });

##### This will produce corresponding "patch" deltas in your HMTL

    //inserted from dynSrc.getPatches
    dynoSrc.apply('my-cool-module', '0.1.2', '...diff...');


## How does it work?

* git diff
  * We are relying on forking out to a child_process.
  * This means the process you are running node with must have access to git.
  * The reason we are not using js-git for this was because that feature was not implemented yet.
* DEV mode FTW
  * dynoSrc eats its own dog food in while developing.  We really felt this was an important feature so this isn't just a build tool for "production".  This way any bugs that come from using this strategy will appear earlier in the dev cycle, rather than in production.  Here is a simple run down of what the middleware will do on every page load.
    * Look at the resources in your cookie
    * Look at every local resource in your repo that needs to be served
    * Determine if there were changes (due to direct file change via a human, or maybe grunt rewrote them)
    * Write the changed file to a scratch area with an epoc timestamp
    * Compute a diff from the last page load
    * Send ONLY the diff

## TODOs

* DEV mode WILL eat your entire HD.  Someone needs to write a thing to clean up all the things :)
* Move sever oriented unit tests out of NodeKnockout repo and into public dynoSrc repo.
* Remove hard dependancy on shelling out to git. 
* Much of this feels like it could be complimented with a Grunt task.
* Write client unit tests against dynoSrc.js ;)
* Bit mask resources in cookie to not store entire resource name
