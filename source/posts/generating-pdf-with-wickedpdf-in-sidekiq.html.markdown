---

title: Generating pdf with wickedpdf in sidekiq
date: 2018-06-14 14:11 UTC
desc: A long title for a short post
tags:

---


Recently, I had to generate invoice pdf in a background job.
There is a [documentation](https://github.com/mileszs/wicked_pdf/wiki/Background-PDF-creation-via-delayed_job-gem) for how to do so, but it seemes overly complicated.

Here is a shorter, more readable version and without unnecessary `class_eval`

```ruby
view = ActionView::Base.new('app/views/resources/', {}, ActionController::Base.new)

html = view.render(file: 'show.html.slim', locals: { resource: resource })

WickedPdf.new.pdf_from_string(html)
```

Returned pdf you can then **upload to S3** or **store in a file.
