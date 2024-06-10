# Hugo Hello World


Welcome to Hugo FixIt! This is your very first post.

&lt;!--more--&gt;

Head to the documentation page linked below for a complete guidence to get started with the [FixIt](https://github.com/hugo-fixit/FixIt) theme.

{{&lt; link href=&#34;https://fixit.lruihao.cn/documentation/&#34; content=&#34;All Documentation - FixIt&#34; title=&#34;documentation of FixIt Theme&#34; card=true &gt;}}

## Quick Start

### Prerequisites

Just install latest version of [Hugo(&gt;= 0.109.0)](https://gohugo.io/installation/) for your OS (Windows, Linux, macOS).

### Clone Template

Clone with your own repository url

```bash
git clone --recursive git@github.com:hugo-fixit/hugo-fixit-blog-git.git
```

### Launching the Site

```bash
# Development environment
hugo server --disableFastRender --navigateToChanged --bind 0.0.0.0
# Production environment
hugo server --disableFastRender --navigateToChanged --environment production --bind 0.0.0.0
```

&lt;details&gt;
  &lt;summary&gt;Start via NPM script&lt;/summary&gt;

  ```bash
  npm install
  # build the blog
  npm run build
  # run a local debugging server with watch
  npm run server
  # run a local debugging server in production environment
  npm run server:production
  # update theme submodules
  npm run update:theme
  ```

&lt;/details&gt;


### Build the Site

When your site is ready to deploy, run the following command:

```bash
hugo
```

For a complete quick start, see this [page](https://fixit.lruihao.cn/documentation/getting-started/).

## Questions, ideas, bugs, pull requests

All feedback is welcome! Head over to the [issues](https://github.com/hugo-fixit/FixIt/issues) or [discussions](https://github.com/hugo-fixit/FixIt/discussions) tracker.


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/hugo-hello-world/  

