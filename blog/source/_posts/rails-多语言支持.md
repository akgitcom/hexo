title: rails 多语言支持
date: 2015-01-12 09:01:42
tags: [rails,多语言]
---
[http://railscasts.com/episodes/138-i18n](http://railscasts.com/episodes/138-i18n)
[http://rails-i18n.org/](http://rails-i18n.org/)
[Locale Examples](http://github.com/svenfuchs/rails-i18n/tree/master/rails%2Flocale)
[http://blog.segmentfault.com/xiongxin/1190000000489534](http://blog.segmentfault.com/xiongxin/1190000000489534)

###i18n.rb:
```ruby
I18n.default_locale = 'zh-CN'
LANGUAGES = {
    'English' => 'en',
    'Chinese' => 'zh-CN'
}
```

###application.rb:
```ruby
I18n.config.enforce_available_locales = false
config.i18n.load_path += Dir[Rails.root.join('my', 'locales', '*.{rb,yml}').to_s]
config.i18n.default_locale = :'zh-CN'
```

>kaminari 多语设置

###zh-CN.yml:
```ruby
zh-CN:
  views:
      pagination:
        first: "« 首页"
        last: "末页 »"
        previous: "« 上一页"
        next: "下一页 »"
        truncate: "..."
```