# RTD Theme Änderungen

```bash
sed -i 's/800px/100%/g' rtd/static/css/theme.css
sed -i 's/pygments_style = default/pygments_style = manni/g' rtd/theme.conf
sed -i 's/border-radius:50px/border-radius:0px/g' rtd/static/css/theme.css
sed -i 's/prev_next_buttons_location = bottom/prev_next_buttons_location = both/g' rtd/theme.conf
sed -i 's/#2980b9/#313641/g' rtd/static/css/theme.css
sed -i 's/#343131/#313641/g' rtd/static/css/theme.css
```
