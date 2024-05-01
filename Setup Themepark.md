# Themepark

For a few trickier apps like qbittorrent and NPM, first run the following to get the theme files installed to /opt/appdata/themepark

```bash
  sudo apt-get install jq curl
  sudo bash -c "$(curl -fsSL https://theme-park.dev/fetch.sh)" /opt/appdata/themepark
```

Themes are listed here with screenshots: https://docs.theme-park.dev/themes/sonarr/

## NPM

Add the following lines to your compose under:

```yaml
     volumes:
      - /opt/appdata/themepark/98-themepark-npm:/etc/cont-init.d/99-themepark
```

You can edit the theme selection by editing line 42 in `98-themepark-nginx-proxy-manager`. The default line is `TP_THEME='organizr'`

## Qbittorrrent

FIRST, log into your existing, working qbittorrent webUI. Go to settings, webui, then add the following under `Add custom HTTP headers`:

```yaml
content-security-policy: default-src 'self'; style-src 'self' 'unsafe-inline' theme-park.dev raw.githubusercontent.com use.fontawesome.com; img-src 'self' theme-park.dev raw.githubusercontent.com data:; script-src 'self' 'unsafe-inline'; object-src 'none'; form-action 'self'; frame-ancestors 'self'; font-src use.fontawesome.com;
```

Then, add the following lines to your compose:

```yaml
environment:
  - TP_HOTIO=true
  - TP_THEME=organizr #or whatever theme you want
volumes:
  - /opt/appdata/qbittorrent/themepark/98-themepark-qbittorrent:/etc/cont-init.d/98-themepark
```

