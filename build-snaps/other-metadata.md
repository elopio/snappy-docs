---
layout: base
title: Adopt info from other metadata sources
---

Some of the values [required by the snapcraft.yaml file](/build-snaps/syntax)
might be already provided by other metadata sources available in the upstream
project.

Instead of duplicating this information, you can declare that a snapcraft part
must parse the metadata file from the source, and adopt that parsed
information. For example, the following `snapcraft.yaml` will parse the file
called `metadata-file` and try to extract from it the `version`, `summary` and
`description` for the snap.

```yaml
name: my-snap-name
adopt-info: part-with-metadata

parts:
  part-with-metadata:
    plugin: dump
    parse-info: [metadata-file]
```
<br>

## Metadata sources

Currently, the only supported source of metadata is appstream.

### Appstream

[Appstream](https://www.freedesktop.org/software/appstream/docs/) is a standard
for metadata of software components. It can provide the `summary`,
`description` and `icon` for the snap, and the `desktop` files for the
apps.

Let's say we have an upstream project with the following appstream file in
`my-app.metainfo.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<component>
  <id>com.example.my-app</id>
  <summary>Single-line elevator pitch for your amazing application</summary>
  <description>
    This is applications's description. A paragraph or two to tell the
    most important story about it.
  </description>
  <icon type="local">assets/icon.png</icon>
  <launchable type="desktop-id">
    com.example.my-app.desktop
  </launchable>
</component>
```
<br>

Then you can adopt this info into your `snapcraft.yaml` like this:

```yaml
name: my-snap-name
adopt-info: my-app

apps:
  my-app:
    command: my-app
    common-id: com.example.my-app

parts:
  my-app:
    plugin: dump
    source: http://github.com/example/my-app.git
    parse-info: [my-app.metainfo.xml]
```
<br>

The resulting snap will take the summary and description from the appstream
file and it will use the referred icon and desktop files.

Note that appstream doesn't use a path to declare the desktop file. It uses
a [Desktop File ID](https://specifications.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html#desktop-file-id). To know which appstream
`desktop-id` corresponds to your app, you must declare the `common-id` of the
app in the `snapcraft.yaml`. snapcraft will search for a parsed appstream file
with the same `id` and extract the `desktop-id`. Then it will search for the
desktop file in the directories  `usr/local/share` and `usr/share` relative to
the part source, following the Desktop File ID rules.
