Welcome to my professional <a href="https://hoxuanvinh.netlify.app/">webpage</a>. I forked and modified it from <a href="https://github.com/qwtel/hydejack">Hydejack</a>. The <a href="https://hovinh.github.io/">Github webpage</a> counterpart can no longer render mathematical formula, hence I simply copy the compiled code and deploy at <a href="https://www.netlify.com/">netlify</a> instead.

# What I like in this template
<ul>
  <li>Simple yet beautiful theme.</li>
  <li>You can write mathematical formula.</li>
  <li>A handful of social icons to pick.</li>
  <li>Suitable to showcase your profile as well as blogging.</li>
</ul>

# How to create yours

1. Clone or download this repository to your local machine.
2. Full documentation can be found <a href="https://hydejack.com/docs/">here</a>. I opt for running it locally on Windows, following this <a href="http://jekyll-windows.juthilo.com/">post</a>, with minor adjustment:
    - Install <a href="https://rubyinstaller.org/downloads/">Ruby</a>. Ruby+Devkit 2.5.X (x64) installer contains both Ruby and Ruby Devkit.
    - Install Jekyll: open Terminal/Command Prompt and input: 
    ```powershell
    gem install jekyll
    gem install bundler:1.16.1
    ```
    - Move to the  downloaded repo, where *_config.yml* is located, open *Gemfile* and modify:
    ```powershell
    gem "jekyll"   # -> run in local machine
    # gem "github-pages", group: :jekyll_plugins # -> run in Github 
    ```
    - In Terminal:
    ```powershell
    bundle install
    jekyll serve --incremental
    ```
    > **NOTE**: If failed, you may try *bundle exec jekyll serve --incremental*.  

    - Open browser, e.g. Chrome, and enter *http://localhost:4000* to the address bar.

    - You can try to modify my page and refresh browser to see the changes take effect. Thanks to *--inceremental* parameter, it usually takes 3 seconds to rebuild the page. 

    > **NOTE**: The auto-generated site is located in *_site* directory. To add new content, it must not be inside this directory. If you add/remove a new page, you should 1) interrupt terminal, 2) delete *_site* directory, then 3) rerun *jekyll serve --incremental* again.  
3. Have fun!
    >**NOTE**: before commiting and pushing to Github, remember to remodify *Gemfile*.

# How my page is organized
- `assets`: contains images used.
- `_data`: contains `authors.yml` showing author profile.
- `blog\_posts`: contains the raw content of new posts. You create a new blog page inside here, but note that the date must be today or earlier, not in the future.
- `about.md`: content of About's page.
- `news.md`: content of News' page.
- `projects.md`: content of Projects' page.
- `_site`: where generated website located. Remember to update the inside `robots.txt`/`sitemap.xml` with `robots_netlify.txt`/`sitemap_netlify.xml` so your website can be indexed by Google. Another way is submitting the web url to https://www.xml-sitemaps.com/ to auto-generate. Do verify via Google Search Console.

Please leave a star if you find this repo helpful.
