---
title: Python Flask_admin
---

> python2.6
> 手把手搭建Web网站

Flask-Admin是一个功能齐全、简单易用的Flask扩展，让你可以为Flask应用程序增加管理界面。它受django-admin包的影响，但用这样一种方式实现，开发者拥有最终应用程序的外观、感觉和功能的全部控制权。

**本文是关于Flask-Admin库的快速入门，手把手带你一起学习。**

## 重点模块

| 模块名                                                       | 简介                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [Flask_admin](https://flask-admin.readthedocs.io/en/latest/) | As a micro-framework, [Flask](http://flask.pocoo.org/) lets you build web services with very little overhead. It offers freedom for you, the designer, to implement your project in a way that suits your particular application. |
| [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/) | Flask-SQLAlchemy is an extension for [Flask](http://flask.pocoo.org/) that adds support for [SQLAlchemy](https://www.sqlalchemy.org/) to your application. It aims to simplify using SQLAlchemy with Flask by providing useful defaults and extra helpers that make it easier to accomplish common tasks. |
| [Flask-BabelEx](https://pythonhosted.org/Flask-BabelEx/)     | Flask-BabelEx is an extension to [Flask](http://flask.pocoo.org/) that adds i18n and l10n support to any Flask application with the help of [babel](http://babel.edgewall.org/), [pytz](http://pytz.sourceforge.net/) and [speaklater](http://pypi.python.org/pypi/speaklater). It has builtin support for date formatting with timezone support as well as a very simple and friendly interface to [`gettext`](http://docs.python.org/library/gettext.html#gettext) translations. |
| [WTForms](https://wtforms.readthedocs.io/en/stable/)         | With WTForms, your form field HTML can be generated for you, but we let you customize it in your templates. This allows you to maintain separation of code and presentation, and keep those messy parameters out of your python code. Because we strive for loose coupling, you should be able to do that in any templating engine you like, as well. |
| [Flask-Login](http://www.pythondoc.com/flask-login/)         | Flask-Login 为 Flask 提供了用户会话管理。它处理了日常的登入，登出并且长时间记住用户的会话。 |

