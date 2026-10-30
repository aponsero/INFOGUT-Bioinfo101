---
title: Contributing
---

# Adding material to this website

All the content is written in Markdown files stored in the `modules/` folder of the GitHub repository.

## Adding a tutorial to an existing module

1. Open the module folder, then its `_posts` subfolder, for example `modules/command-line/_posts/`.
2. Create a new file named `YYYY-MM-DD-short-name.md`. The date is not displayed: it only sets the order of the tutorials in the sidebar. Each module starts with `1980-01-01-programme.md` (the module overview), and tutorials use `2000-01-01`, `2000-01-02`, and so on.
3. Start the file with a title:

   ```
   ---
   title: Navigating the file system
   ---
   ```

4. Write the tutorial in Markdown below the title.
5. Put slides, datasets and scripts in the `data/` folder of the module, for example `modules/command-line/data/`, and link to them with:

   {% raw %}
   ```
   [Download the slides]({{ site.baseurl }}/modules/command-line/data/slides.pdf)
   ```
   {% endraw %}

6. Add the tutorial to the list on the module overview page, and update the module `status` in `_data/modules.yml` if needed.

## Adding a new module

1. Create a folder `modules/<module-name>/_posts/` with an overview file `1980-01-01-programme.md` (copy one from another module).
2. Add the module name to the `modules` list in `_data/course.yml`, at the position where it should appear in the menu.
3. Add an entry for the module in `_data/modules.yml` (title, summary, authors, status).

## Advanced material

Advanced tutorials go on the separate advanced-level website, not here.
