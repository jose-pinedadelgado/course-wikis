# Output Structure

## Generated Wiki

```
<course-slug>/
├── mkdocs.yml              # Auto-generated site config
├── pyproject.toml           # Python project config
├── .python-version
└── docs/
    ├── index.md             # Course overview
    ├── contributing.md      # Standard template
    └── modules/
        ├── week-01/         # One folder per Canvas module
        │   ├── index.md     # Module overview
        │   ├── lecture.md   # From .pptx
        │   ├── activity.md  # From .docx
        │   └── code.md      # From .sql
        └── week-02/
            └── ...
```

## Config Generation

`mkdocs.yml` is generated from the module tree:

```python
def generate_mkdocs_yml(course: Course) -> str:
    nav = []
    for module in course.modules:
        section = {module.display_name: []}
        for page in module.pages:
            section[module.display_name].append(
                {page.title: page.relative_path}
            )
        nav.append(section)
    
    return render_template("mkdocs.yml.j2", 
        site_name=course.title,
        nav=nav,
        theme_color=course.color or "indigo"
    )
```

## Framework Compatibility

Chalq generates standard Markdown + `mkdocs.yml`. Both MkDocs Material and Zensical consume the same input:

| Feature | MkDocs Material | Zensical |
|---|---|---|
| `mkdocs.yml` format | ✅ Same | ✅ Same |
| Admonitions | ✅ | ✅ |
| Code highlighting | ✅ | ✅ |
| Dark mode | ✅ | ✅ |
| Search | ✅ | ✅ |
| Mermaid diagrams | ✅ | ✅ |

**Switching framework** is a one-line dependency change:

```toml
# MkDocs Material
[tool.uv]
dev-dependencies = ["mkdocs-material>=9.6"]

# Zensical
[tool.uv]
dev-dependencies = ["zensical>=0.0.23"]
```

## Dual-Repo Publishing

Chalq outputs go to two repositories:

| Repo | Visibility | Purpose |
|---|---|---|
| `jose-pinedadelgado/chalq` | Private | Source of truth, may contain student data |
| `jose-pinedadelgado/course-wikis` | Public | Published wikis, sanitized |

The `chalq publish` command handles the copy, commit, and push.
