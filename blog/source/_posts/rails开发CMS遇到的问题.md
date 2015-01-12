title: rails开发CMS遇到的问题
date: 2015-01-12 09:43:33
tags: [rails,cms]
---

###关联排序并取出第一个IMAGE

> uploader用了carrierwave的七牛云,如果不用：
<%= b.photos.order(sort: :asc).pluck(:image).pop || “blank.jpg” %>
b.photos.order(sort: :asc) 你的
.pluck(:image)，取出image字段
.pop，取出第一个，如果为空，返回nil
|| blank，默认图片

```rails
<% @products.each do |b|%>
<li>
<a href="/products/<%=b.id%>.html">
<%= image_tag( b.photos.order(sort: :asc).map(&:image_url).pop) %>
</a>
<%= link_to truncate(strip_tags(b.title), length: 60, escape: false),
                            {:controller => "products",
                             :action => "show",
                             :id => "#{b.id}",
                             :format=>"html"
                            } %>
</li>
<% end %>
```

###截取字符串包括”…”加字符数量
```rails
truncate(strip_tags(b.title), length: 60, escape: false)
```