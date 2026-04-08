# dblindanon.github.io

To add a page, create an HTML file in `pages/` with front matter specifying `layout: default`, `title`, and `permalink` (see existing pages for examples). Then add a corresponding `<div class="menu-item">` entry in `_layouts/default.html` to make it appear in the sidebar. Static assets (images, CSS) go in `assets/images/` and `assets/css/` respectively.

To test the page, setup the environment with `bash setup_env.sh` and then activate your environment `source source_me`.

Once you've activated the environment, you can serve your website using the bash command `serve`. That will start a local webserver at `http://localhost:4000`, which you can connect to in your browser. If you're running this remotely, you'll need to forward port `4000`.
