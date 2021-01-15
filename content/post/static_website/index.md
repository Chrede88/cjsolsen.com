+++
title = "Creating and deploying a static website"
subtitle = "Using Hugo and Amazon Web Services (AWS) or Github"
date = 2020-10-08T15:00:00-08:00
draft = false

# List authors (if not me!)
authors = []

# Add code highlight
highlight = true

summary = "Use Hugo to easily build a nice website in a couple of hours and deploy it using AWS or Github."

# Tags and categories
# For example, use `tags = []` for no tags, or the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = ["Website","Hugo","AWS","Github Pages"]
categories = []

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["deep-learning"]` references
#   `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects = []

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
[image]
  # Caption (optional)
  caption = ""

  # Focal point (optional)
  # Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
  focal_point = "Center"
+++

Building a website can easily seem like quite an overwhelming task. Even if you already know a bit of HTML, CSS and Java script. A simple statically typed website can be a mouthful for most people. Here statically typed means that the website doesn't have any elements that needs computing power and is therefore both cheaper to host and easier to build. Static websites are much simpler, but also have some limitations. Luckily, there are great tools that can help anyone setting up a simple website. Follow this link if you want to read more about static sites: [An introduction to static site generators](https://davidwalsh.name/introduction-static-site-generators).

# Table of content
1. [Using Hugo to build a static website](#usinghugo)
    1. [Installing Hugo](#install)
    2. [Choosing a theme](#theme)
    3. [Editing your website](#edit)
    4. [Building the website](#build)
2. [git interlude](#git)
    1. [Deploying on Github](#deploygithub)
3. [Deploying using Amazon Web Services](#aws)
    1. [Storing files in S3 buckets](#s3)
    2. [Setting up a Content Delivery Network (CDN) - CloudFront](#cloudfront)
    3. [Buying and setting up a domain using Route 53](#route53)
    4. [Automation with Github Actions](#autos3)

# <a name="usinghugo"></a> Using Hugo to build a static website
[Hugo](https://gohugo.io) is a static website generator build using [Go](https://golang.org). It outperforms most static site generators and there is a large set of pre-build themes to choose from. Of course it's also possible to build a website from scratch, but starting from a theme can reduce the time needed to have a fully functioning website to less than 30min! This site uses the [Academic](https://themes.gohugo.io/academic/) theme. Don't be alarmed, you don't really need to know anything about Go, HTML or CSS. Although a bit of HTML knowledge would be advantageous.

## <a name="install"></a> Installing Hugo
Firstly we need to install Hugo and possibly some prerequisites. If you are using macOS or Linux, you should install [Homebrew](https://brew.sh). Homebrew is a package manager, which will make your life a lot easier!

### macOS or Linux
Run the following to install Homebrew (in a Terminal window):
``` bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```
Make sure you have the latest version of Homebrew:
``` bash
brew update && brew upgrade
```
We are now ready to install Hugo, Go and git. If you already have git and/or Go installed, then leave them out.
``` bash
brew install git golang hugo
```
Hugo relies and Go to build your website files and we need git so we can interact with Github (or another online git repository manager). I will assume you are using [Github](https://github.com) in this tutorial.

### Windows
If you are using Windows, you should install [Scoop](https://scoop.sh), using Powershell.
``` bash
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
iwr -useb get.scoop.sh | iex
```
Now install Hugo and prerequisites, leaving out stuff you already have.
``` bash
scoop install git openssh go hugo-extended
```
### All operating systems
Test that Hugo is correctly installed:
``` bash
hugo version
```
Output should be something like this:
``` bash
Hugo Static Site Generator v0.74.3/extended
```

## <a name="theme"></a> Choosing a theme
Most good themes have nicely complied documentation, which should get you started.
If you want to use the [Academic](https://themes.gohugo.io/academic/) theme, as is used for this website, run the following command in Terminal or Powershell at the location you want to store your website files.
``` bash
git clone https://github.com/wowchemy/starter-academic.git
```
If you don't like Academic, check out the many other nice themes available at [Hugo themes](https://themes.gohugo.io) and clone your favourite theme to your local machine.

## <a name="edit"></a> Editing your website
It is now time to add some content to your new website. But first let us take a look at the folder/files structure used by Hugo. Any new site will have more or less the same structure.
``` text
. <site name>
├── archetypes
├── config.toml
├── content
├── data
├── layouts
├── static
└── themes
```
The most important file is the `config.toml` (or `.yaml`). Most of the main info about the website is stored here. Alternatively the theme might use a configuration folder containing more than one config file. No need to worry if you don't know `.toml` or `.yaml` syntax! They are both very similar and very simple. I will use `.toml` throughout this tutorial, but `.yaml` will be as simple. Let us take a look at a small (minimal) config file.
``` ini
baseURL = "https://yoursite.example.com/"
title = "My Hugo Site"

[params]
  AuthorName = "Jon Doe"
  GitHubUser = "spf13"
  ListOfFoo = ["foo1", "foo2"]
  SidebarRecentLimit = 5
  Subtitle = "Hugo is Absurdly Fast!"

[permalinks]
  posts = "/:year/:month/:title/"
```
A real configuration file will contain much more, but the syntax is easy to understand. Right now the title of our website is `My Hugo Site`. If we want to change that, simply change the content of title key: `title = "New website title"`. Simple!:smiley:
While you are developing the website you can leave the `baseURL` as is, but it needs to reflect the actual `URL` once you are ready to deploy your site. We will get back to that later.

If you want to take a look at your website as you are adding content or changing parameters, you can run a local HTTP server right from a Terminal (or Powershell) window.
``` bash
hugo server
```
This will start a local HTTP server running at `http://localhost:1313`. Open a browser and navigate to this address. You will likely see an empty or almost empty website, we haven't added any content just yet:wink:

Once you add more stuff to the website, the HTTP server will pick up the changes and render the site on the fly. Press `Ctrl + c` to stop the HTTP sever.

Apart from the config file(s), the important folders are `content`, `static` and `theme`. The `content` folder will hold blog posts and other larger content. The `static` folder is where images used on the main page will be stored. The `theme` folder is pretty self explanatory, it holds all the files necessary for the chosen theme. Take a look at [this link](https://gohugo.io/getting-started/directory-structure/) for more info on the folder structure used by Hugo.

## <a name="module"></a> Hugo modules
Modules are supported as of Hugo v0.56.0. Modules are like packages in Python or modules in Go (which Hugo modules are build upon). You can use modules to import certain blocks of code that can help you build your website or even whole themes can be imported as modules. If you have chosen to work with the Academic theme I linked to under [Choosing a theme](#theme), you will notice that the theme folder is missing! This is because it is imported as a module on compile. If you look in the main config file (./config/_default/config.toml) you will see a `[module]` section at the very bottom. This instructs Hugo to import the theme. If you are not using Netlify (this tutorial does not rely on Netlify) you can out comment the second `[[module.imports]]`.

Whenever you build your website Hugo will download the version specified in the file called `go.mod`, or load it from cache. This ensures that you will always get the same version of the theme unless you update the version pointer in the file. This also makes it extremely easy to update the theme to a newer version.

## <a name="build"></a> Building the website
Once you are ready to compile the HTML files that will make up your website, navigate to the top level of your local website folder tree and execute the following:
``` bash
hugo
```
This will build the files needed in a new folder called `public`. Every time you build your website the files in `public` will be updated and it is these files you need to upload to where ever your website is hosted.

# <a name="git"></a> git interlude
At this point you should consider creating a remote repository for your website. I use [Github](https://www.github.com), it's a great platform and you can actually host your website on Github completely free. Publishing your files may also help others in making their own website. Open source people!:smiley:

If you are unsure about how to use git, take a look at [this presentation](https://chrede88.github.io/git-presentation/) I have put together. The presentation goes through all steps from installing git to collaborating with others. Please note that the presentation layout is two dimensional, look for arrows in the bottom right corner or press `esc` to get an overview.

## <a name="deploygithub"></a> Deploying on Github
As mentioned above, you can deploy your website on Github for free. If you choose this option, there is no reason to read the sections about Amazon Web Services (AWS) that follow this section.

Note that your website `URL` is going to be `https://<username>.github.io/<repository>/`, where `<username>` is your Github username and `<repository>` is the name of the website repository on Github. Remember to set the `baseURL` in the config file, otherwise most of your website will not work!

We are going to take advantage of Github's "Pages", called `gh-pages`, functionality to deploy the website. `gh-pages` is meant as an easy way of making code documentation, but it can serve any kind of HTML. To make it even easier I'm going to tell you how to setup an automation workflow. This will automatically update the website each time Github detects a push to the master branch.
Before starting the setup, I'll assume that you have pushed your master branch to Github and that the `baseURL` parameter in the configuration reflects the correct URL provided by Github.

### <a name="actiongithub"></a> Setting up the Github automation
Github automations can be found under the `Actions` tab, when viewing your repository. Github will present a bunch of ready-to-use workflows, but we are going to write our own.

{{< figure src="actions.png" title="Click on `set up a workflow yourself ->` " >}}

Github will populate the new workflow file with some standard input. Notice that Github uses `.yaml` files for their workflows. Key/value pairs are used just like in `.toml` files. The indents (2 spaces) are very important and the workflow will not work if they are messed up. Go ahead and replace the standard input with the following:
``` yaml
# Give the workflow a name
name: Deploy to gh-pages

# Controls when the action will run. Triggers the workflow on push request
# events to the master branch.
on:
  push:
    branches: [ master ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  Build_and_Deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.74.3'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

The first part defines when this workflow should run. Change the name of the branch from `master` to something else if you want to work off another branch. The next section defines what jobs to run when executing the workflow. There is only one job, `Build_and_Deploy`, and there are four steps in this one job. The first step checks out the master branch so the automation can access the files. Next step installs Hugo on the virtual machine running the workflow and the third step builds your website and puts the compiled files into the public folder. The last step creates (if it doesn't exists) a branch called `gh-pages` and places the files from the public folder into this branch.

### <a name="golangGithub"></a> Modules and Go
The content of the workflow file I posted above needs an addition if your website relies in Hugo modules, like the Academic theme.
Importing and building Hugo modules requires Go as well as Hugo. The workflow file should contain an extra step:
``` yaml
# Give the workflow a name
name: Deploy to gh-pages

# Controls when the action will run. Triggers the workflow on push request
# events to the master branch.
on:
  push:
    branches: [ master ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  Build_and_Deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.74.3'
          extended: true

      - name: Setup Go
        uses: actions/setup-go@v2
        with:
          go-version: '1.15' # Import the Go version specified in go.mod

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```
Notice the extra step called `Setup Go`. This installs Go on the virtual machine, so it can be used to compile the theme. Without it, the workflow will fail!

### <a name="setingsgithub"></a> Setting the Github Pages source
The last step is to inform Github where to look for the source for the website. Under the `Settings` tab, scroll down to the `Github Pages` section and set the source to the newly created branch `gh-pages`. Github will now start building the website and will post a small messages when your website is ready. It usually takes less than 2 min.

{{< figure src="gh-pages.png" title="Choose the `gh-pages` branch as the source." >}}

Lastly check out the storage and bandwidth limits imposed by Github on these sites: [Guidelines for using github pages](https://docs.github.com/en/free-pro-team@latest/github/working-with-github-pages/about-github-pages#guidelines-for-using-github-pages). For most small personal websites, these limits are not a concern. Note that e-commerce sites etc. are prohibited.

# <a name="aws"></a> Deploying using Amazon Web Services
Using Amazon Web Services (AWS) is not free, but offer quite a few advantages over Github or other free services like GitBucket. You shouldn't really worry about the price. I pay around 53 US cents per month for the services I use to host this site. Choosing AWS will not by any means break your bank. The reason we can get away with such a cheap option is because we don't need any computing power - the magic of static websites:smiley: You will also need to buy a domain, which will set you back $10-$12 a year, for the most common top level domains (TLD), like `.com`. So all-in-all you are looking at max $20 per year.

Amazon has a free tier, that provides users with free access to most of their services and a years worth for the rest. Follow this link to [sign up](https://aws.amazon.com/free/). With the free tier your website will be completely free (apart from the domain) the first 12 months! After the first year you will have to pay for the files stored in your S3 buckets and the amount of traffic to your site, but the prices are almost non existing. Amazon charges $0.023 per GB of storage up to the first 50TB and $0.01 per 10000 HTTPS requests, all per month. Lastly you will have to pay $0.50 for the DNS routing, also per month. All-in-all it's super cheap!

As an fun little aside, think about how much you pay for your Dropbox/iCloud/GDrive storage compared to S3. The only setback is that AWS doesn't have a nice interface app. But if you are comfortable with commmand-line interfaces, this might be a good and very cheap alternative to the standard cloud storage options.

Let us get started setting up the AWS services you need to host your website.

## <a name="s3"></a> Storing files in S3 buckets
Once logged into AWS, navigate to the Management Console, from where you can access all the services available. Amazon's cloud storage service is called S3 (Simple Storage Service) and the single containers are called buckets. You will need to create two buckets, one that will hold your website files and one that will redirect to the first one. This is necessary in order to make sure users can access your site with and without adding "www" to the URL, i.e. `example.com` and `www.example.com`. The bucket names should reflect the two URLs you want your website to be served at, i.e. `example.com` and `www.example.com`.

It would be smart to check that the domain you want to buy actually is available and not taken already, at this point. Otherwise, you might have to redo everything. Head over to the service called Route 53. Under "Domains", check if the domain you want is available.

{{< figure src="buckets.png" title="Create two buckets with the names reflecting your website URL." >}}

Create the first bucket, which will contain the files. This bucket should have the name **with** www. Select a region close to you (it will not have any real impact). In the next section untick `Block all public access`, so the outside world can access your website. Leave the rest of the settings as default.

{{< figure src="statichosting.png" title="Setup the bucket as a Static website host." >}}

Next setup the bucket as a static website host under `properties`. This gives the bucket an endpoint URL so the outside world can access your files via a browser. Set the index document to `index.html` and the error document to `404.html`.

{{< figure src="bucketsetup.png" title="Input the index and error documents." >}}

The last step is to setup a bucket policy, which restrict the access to the bucket to read only. Add the following to the bucket policy under `permissons`. Change the `Resource` to the `arn` of your bucket.

``` json
{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Sid": "AllowPublicRead",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::www.cjsolsen.com/*"
        }
    ]
}
```
The important line is `Action`, where access is restricted to `GetObject`.

Next create the second bucket with the name **without** www. This bucket should also be setup as a Static website host, but this bucket should redirect to the first bucket.

{{< figure src="secondbucket.png" title="Setup the second bucket to redirect to the first bucket." >}}

This bucket is not accessed directly, so no need to setup a policy.

At this point your website is actually online! If you navigate the to endpoint URL I mentioned above. It will be something like `http://<bucketname>.s3-website-us-east-1.amazonaws.com`, details depending on which location you chose for your buckets. Write this address down, as you will need it when setting up the Content Delivery Networks.

Now you could just use this URL, but there is a few things to consider. Firstly, it is not a very easy URL to remember. Secondly, the connection is over HTTP and not HTTPS which is **not** secure. This is bad on its own, but it might also prevent some browsers from displaying your website without extra user input. Thirdly, if users far away from the location of your buckets try to access your website, they might have to wait quite long for the content to load.

The first two issues will be fixed later. First I will tell you how to setup a Content Delivery Network (CDN) using CloudFront. This will make sure your website loads fast and securely for anyone, anywhere in the world.

## <a name="cloudfront"></a> Setting up a Content Delivery Network (CDN) - CloudFront
A Content Delivery Network is in an essence a bunch of server each holding a cached version (a hardcopy) of your website. The servers in the network are physically distributed across the world and can therefore serve users around the world faster than if they all had to request your website from one specific server.

{{< figure src="cdn.png" title="A Content Delivery Network can serve users more efficiently than a single server." >}}

For a small website it might not make a huge difference, but if you have a website with many large images or high resolution videos, it can reduce the loading time to less than 1s from many seconds. CloudFront also have some build-in mechanisms that will prevent DDos (distributed-denial-of-service) attacks. DDos attacks tries to flood the server with requests so that it can not serve the "real" requests. I will not pretend that my website is so important that a DDoS attack will have any actual impact, but I do pay Amazon when people visit my site. So if someone sends 1 billion requests to my website every day I will end up with a substantial bill at the end of the month!

Enough about CDNs, let us setup up a couple of networks. You will need to create two distributions, one for each of the two S3 buckets.

{{< figure src="cloudfront.png" title="Create two distributions, one for each of the two S3 buckets." >}}

You need to create Web distributions. Your two S3 buckets should show up when you click on `Origin Domain Name`. For some strange reason the names provided in the dropdown menu are **NOT** correct. You will find the correct address under `S3 -> <bucketname> -> properties -> Static website hosting -> End point`. Don't include `http:\\` in the address. Select `redirect HTTP to HTTPS` under Viewer Protocol Policy.

{{< figure src="createCF.png" title="Remember to select 'Redirect HTTP to HTTPS'." >}}

Under Distribution Settings input `index.html` under Default Root Object. You only need to do this for the "www" distribution, leave it blank for the redirect distribution. Stick to the default SSL certificate for now. But you will have to update that once you have own.

{{< figure src="distributionsettings.png" title="Input 'index.html' under Default Root Object." >}}

Save the distribution and create another one for the second S3 bucket.

Your newly created CloudFront distributions each have a domain name. Just like with the S3 bucket endpoint domain name, you should now be able to point your browser to these two distribution addresses and see your website. This time you should be redirected to a HTTPS connection and you should be able to see the SSL certificate as well.

## <a name="route53"></a> Buying and setting up a domain using Route 53
It is now time to buy a personal domain so your website can have a URL that is easy to remember. And once you have your own domain you have to use DNS (Domain Name System) to route your traffic from your domain to the two CloudFront distributions.

First things first! Head over to Route 53 and press `Register Domain`. Now pick your domain name, extension and hit check. Amazon will tell you if the domain is available. Once you have found a domain you like, it is time to enter your contact information that will get attached to the domain. All there is left now, is to pay and you officially own a domain:smiley:

You will also need a SSL/TLS certificate for your domain. To start that process head over to AWS Certificate Manager. Click `Get Started` or `Request a certificate` if you don't land on the start page.

Firstly enter the domains you want the certificate to cover. I would recommend covering all possible subdomains so you can easily expand your website in the future without having to mess with the certificate. To do this add both the shortform URL e.g. `example.com` and a second with a subdomain wildcard, e.g. `*.example.com`. You might not know, but "www" is actually a subdomain and is therefore covered by the wildcard. If in the future you want to add some content to `blog.example.com`, you could do that without having to update your certificate.

Next you will have to pick a validation method. You don't have an email service setup, so your only option is DNS validation. This process can be a little bit confusing. Here is a link to Amazon's [user guide](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-validate-dns.html), scroll down to the section called `Validate with DNS`. It should not be too hard to follow along. Once you have finished the validation, which is needed in order to prove that you have control over the domain, you have to wait an hour or two for the process to finish. Once it is done the status says `Issued` in green.

{{< figure src="certificate.png" title="SSL/TLS certificate issued." >}}

You will notice that I did not follow my own advice!:facepalm:

Once you have your certificate, you need to update the CloudFront distributions. Go back to CloudFront and click on the ID tag of each distribution. Hit edit under the General tab and change the SSL/TLS certificate to a custom one. You should be able to pick the new certificate from the dropdown menu. You also have to enter the CNAME, either with or without "www", depending on which distribution you are editing.

{{< figure src="updatecertificate.png" title="Choose your new certificate and enter the CNAME." >}}

Last thing to do is to setup the DNS routing. Head back to Route 53. Click on Hosted zones in the left menubar and click on your domain. Now, create a new record, choose "simple routing" and click on "define simple record".

Enter "www" in the name field. The type should be "A", for normal IPv4 routing. Under "Value/Route traffic to", pick "Alias to CloudFront distribution" and pick the distribution from the dropdown menu. You should only be able to pick the "www" distribution. Leave the TTL at 300s and click "Define simple record".

{{< figure src="DNSroute.png" title="Setup two simple records pointing at your two CloudFront distributions." >}}

Setup a second route for the other CloudFront distribution in the same way, this time leaving the name field blank.

DNS can take a bit of time, but usually not more than an hour. Route 53 will not tell you when it is done, so you just have to try to open your website a couple of times until it shows up.

## <a name="autos3"></a> Automation with Github Actions
Just like when deploying with Github, [Setting up the Github automation](#actiongithub), we can automate the process when updating your website. The automation will generate the public files with Hugo, push them to your S3 bucket and invalidate all cached website data on your CDNs. It is not necessary to invalidate all cached data, but it can otherwise take up to 24h before all edge locations are updated. Invalidations are free as long as they are kept under 1000 per month. I am not 100% sure if the count is per file or per invalidation request. Just keep it in mind!

Before setting up the Github Action, we need to make a new user on AWS. AWS let us create users called IAM. These users can be heavily restricted, so they can only interact with a very specific subset of services. This is important because we otherwise give our Github Action admin rights. To setup a IAM user, head over to the IAM Management Console, from the main AWS Console.

{{< figure src="IAMuseradd.png" title="Add a new IAM user in the IAM Management Console." >}}

Give your users a descriptive name and select "Programmitic access" for the account type. Select "Attach existing policies directly" and click "Create policy". Add the following policy to the `JSON` tab:

``` json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "GitHubActionsPolicy",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:DeleteObject",
                "s3:ListBucket",
                "s3:GetBucketPolicy",
                "s3:PutBucketPolicy",
                "cloudfront:CreateInvalidation"
            ],
            "Resource": [
                "arn:aws:cloudfront::857922532360:distribution/<distributionID>",
                "arn:aws:s3:::www.cjsolsen.com",
                "arn:aws:s3:::www.cjsolsen.com/*"
            ]
        }
    ]
}
```
Add your CloudFront distribution ID to the CloudFront resource line, it should be the "www" distribution. And change the bucket name to the name of your bucket that contains the website files.
Skip the tags section and create the user.

Next we need to add an access key to the user. Open your newly created user and navigate to the Security credentials tab and click "Create access key". Download the .csv file containing the credentials, we need to add them to the Github Action.

We are now done with AWS, next head over to your Github repository. Under Settings/Secrets add two secrets, one for the AWS key and one for the password. Call them something that makes sense, so you know what they are.

Now we are ready to create the automation action! Under the Action tab, hit "set up a workflow yourself ->".

{{< figure src="actions.png" title="Click on `set up a workflow yourself ->` " >}}

Github will populate the action with some default stuff, replace it with the following:
``` yaml
# This is a basic workflow to help you get started with Actions

name: Build site and deploy to S3 bucket

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the master branch
on:
  push:
    branches: [ master ] # <-- Change to another branch, it you don't want to work on another branch

jobs:
  Build_and_Deploy:
    runs-on: ubuntu-18.04
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - uses: actions/checkout@v2
      with:
        submodules: true  # Fetch Hugo themes (true OR recursive)
        fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

    # Intall Hugo
    - name: Setup Hugo # <-- installing Hugo on virtual machine
      uses: peaceiris/actions-hugo@v2
      with:
        hugo-version: '0.74.3'
        extended: true

    # Builds cjsolsen.com repo
    - name: Build Hugo # <-- Biulding website and adding files to ./public
      run: hugo --minify

    # Deploys built website to S3
    - name: Deploy to S3 # <-- use build-in deploy function to add files to S3 and create invalidations
      run: hugo deploy --force --maxDeletes -1 --invalidateCDN
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_KEY }} # <-- Change to the name you gave the AWS key secret
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_PASS }} # <-- Change to the name you gave the AWS passwd secret
```

Remember to change the last two lines, so they reflect the names you gave your AWS secrets. Give the workflow some reasonable name and commit it to the master branch.

Just like I explained under [Modules and Go](#golangGithub), you will need an extra step in the workflow file if your website relies on Hugo modules. You need to add extra step that installs Go on the virtual machine. Here is the updated workflow file you will need:

``` yaml
# This is a basic workflow to help you get started with Actions

name: Build site and deploy to S3 bucket

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the master branch
on:
  push:
    branches: [ master ] # <-- Change to another branch, it you don't want to work on another branch

jobs:
  Build_and_Deploy:
    runs-on: ubuntu-18.04
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - uses: actions/checkout@v2
      with:
        submodules: true  # Fetch Hugo themes (true OR recursive)
        fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

    # Install Hugo
    - name: Setup Hugo # <-- installing Hugo on virtual machine
      uses: peaceiris/actions-hugo@v2
      with:
        hugo-version: '0.74.3'
        extended: true

    # Install Go
    - name: Setup Go # <-- installing Go on virtual machine
      uses: actions/setup-go@v2
      with:
        go-version: '1.15' # Import the Go version specified in go.mod

    # Builds cjsolsen.com repo
    - name: Build Hugo # <-- Biulding website and adding files to ./public
      run: hugo --minify

    # Deploys built website to S3
    - name: Deploy to S3 # <-- use build-in deploy function to add files to S3 and create invalidations
      run: hugo deploy --force --maxDeletes -1 --invalidateCDN
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_KEY }} # <-- Change to the name you gave the AWS key secret
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_PASS }} # <-- Change to the name you gave the AWS passwd secret
```

Once the commit is done, the workflow should start. You can follow along under the Action tab. The workflow should fail when it reaches the last step! We still haven't told it which S3 bucket or CloudFront distribution we want it to interact with. To do that you need to add some more info to the `config.toml` file. Add this to the end of the file:
``` ini
# Deployment details
[deployment]

  [[deployment.targets]]
    name = "cjsolsen.com"
    URL = "<bucketURL>"

    # If you are using a CloudFront CDN, deploy will invalidate the cache as needed.
    cloudFrontDistributionID =	"<distributionID>"

  [[deployment.matchers]]
    # Cache static assets for 20 years.
    pattern = "^.+\\.(js|css|png|jpg|gif|svg|ttf)$"
    cacheControl = "max-age=630720000, no-transform, public"
    gzip = true

  [[deployment.matchers]]
    pattern = "^.+\\.(html|xml|json)$"
    gzip = true

```
The deployment.targets describe the name of the deployment and the URL should point to your bucket, e.g. `s3://bucketname?region=us-east-1`. If the CloudFront distribution ID is present Hugo will invalidate your CDN caches as needed. You need to point to your main CDN, the one pointing at the "www" S3 bucket.

Congratulations, you now have a website and a fully automated distribution pipeline🎉🎉🎉
