# Portfolio — Jekyll template

## Setup

```bash
bundle install
bundle exec jekyll serve
```

Visit `http://localhost:4000`.

## File structure

```
_config.yml           ← site settings, about text, skills, social links
_layouts/
  default.html        ← nav + footer wrapper used by all pages
  project.html        ← detail page layout (extends default)
_projects/
  template.md         ← annotated template — copy this for new projects
  katharsi.md         ← example filled-in project
  *.md                ← one file per project
assets/
  css/main.css        ← all styles
  images/             ← cover images for projects
index.html            ← index page (auto-populates from _projects/)
```

## Adding a new project

1. Copy `_projects/template.md` to `_projects/your-project-name.md`
2. Fill in the front matter fields (all documented in the template)
3. Write the body content in Markdown below the `---`
4. Drop a cover image in `assets/images/` and set the `image:` field
5. Set `order:` to control where it appears on the index

The index page card is driven entirely by front matter — you never
touch `index.html`.

## Hiding a project

Set `featured: false` in the project's front matter. The detail page
still exists and is accessible by URL, it just won't show on the index.

## Front matter reference

| Field                   | Required | Description                                            |
| ----------------------- | -------- | ------------------------------------------------------ |
| `title`                 | yes      | Project name                                           |
| `order`                 | yes      | Sort order on index (lower = higher)                   |
| `summary`               | yes      | 1-2 sentence description for card + hero               |
| `status`                | no       | Badge text (e.g. "Live on Steam")                      |
| `status_color`          | no       | Badge color: `open`, `wip`, `private`, or blank (blue) |
| `tags`                  | no       | Tech tags list                                         |
| `image`                 | no       | Path to cover image                                    |
| `image_alt`             | no       | Alt text for cover image                               |
| `stats`                 | no       | List of `{value, label}` pairs for stat row            |
| `link_detail_label/url` | no       | Primary CTA button (e.g. Steam page)                   |
| `link_source_label/url` | no       | Source link (e.g. GitHub)                              |
| `link_extra_label/url`  | no       | Third link (e.g. blog post)                            |
| `featured`              | no       | Set to `false` to hide from index                      |
