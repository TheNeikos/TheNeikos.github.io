---
date: 2018-03-22 07:50:34
title: Using sprocket-rails in production
draft: true
---

I'm sure we have all had a similar moment:

We need to load an asset in our application. Instead of putting in the full
file-system path, we decide to let sprockets handle it. This works fine, and we
push it to our production or staging environment...

Which then breaks, and complains about `Rails.application.assets` being `nil`.

Luckily there is a way around this:

```ruby
def find_asset path
    Rails.application.assets_manifest.find_sources(path).first
end
```

This works both in development and production!
