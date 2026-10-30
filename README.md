# INFOGUT-Bioinfo101: self-study course, beginner level

Self-study tutorials for the analysis of microbiome and metabolomics data from in vitro colon models, developed within COST Action CA23110 (INFOGUT), Working Group 4. The material comes from the Third INFOGUT Training School (Madrid, September 2026).

The website is built with [Course in a Box](https://course-in-a-box.p2pu.org) (Jekyll) and published with GitHub Pages.

## Where things are

| What | Where |
|---|---|
| Home page text | `index.md` |
| Course title and menu order | `_data/course.yml` |
| Module titles, summaries, authors, status | `_data/modules.yml` |
| Module content | `modules/<module-name>/_posts/` |
| Slides, datasets, scripts | `modules/<module-name>/data/` |
| Site settings (base URL, logos, link to the advanced site) | `_config.yml` |
| INFOGUT colours and styles | `_sass/_infogut.scss` |

How to add a tutorial or a module is explained on the website, in About > Contributing (`modules/about/_posts/2000-01-04-contributing.md`).

## Publishing

On GitHub, go to Settings > Pages and publish from the main branch, root folder. The `baseurl` in `_config.yml` is set to `/INFOGUT-Bioinfo101`, the repository name.

## Running locally

```
bundle install
bundle exec jekyll serve
```

Then open http://localhost:4000/INFOGUT-Bioinfo101/ in a browser.

## Licence

Course materials: [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). Website template (Course in a Box, P2PU): MIT licence, see `LICENSE`.
