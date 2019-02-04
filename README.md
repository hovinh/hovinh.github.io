Welcome to my professional <a href="http://hovinh.github.io">webpage</a>. I forked and modified it from <a href="https://github.com/qwtel/hydejack">Hydejack</a>.

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
    - Open browser, e.g. Chrome, and enter *http://localhost:4000* to the address bar.

    - You can try to modify my page and refresh browser to see the changes take effect. Thanks to *--inceremental* parameter, it usually takes 3 seconds to rebuild the page. 

    > **NOTE**: The auto-generated site is located in *_site* directory. To add new content, it must not be inside this directory. If you add/remove a new page, you should 1) interrupt terminal, 2) delete *_site* directory, then 3) rerun *jekyll serve --incremental* again.  
3. Have fun!
    >**NOTE**: before commiting and pushing to Github, remember to remodify *Gemfile*.

Please leave a star if you find this repo helpful.
