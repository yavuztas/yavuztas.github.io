---
layout: about
title: About
permalink: /about/
image: "/assets/article_images/about/about.jpg"
image2: "/assets/article_images/about/about-mobile.jpg"
---
>The problem does not lie in technology but design. I am a true believer in code quality. And I believe clear code and correct design with enough time bring absolute quality to software development.

Let me tell you about myself...

In short, I am a passionate software developer who loves **Java** technologies.

For more details about me, I am a hard-working, innovative software developer with active field experience since 2009, mostly specialized in back-end enterprise technologies using Java and Java EE. I have practical knowledge about solving problems using various design patterns and OOP techniques. I am highly skilled at Spring Framework, Spring Security, ORM frameworks especially JPA, Hibernate and a variety of backend related technologies as well as experienced about UI/UX technologies both server-side rendered like JSF, Vaadin and client-side rendered like Javascript, Jquery and frontend MVC frameworks like Angular, Aurelia. Besides, I am a strong follower of test-driven development, issue tracking, continuous integration, and source code management.

Among my recent achievements, I have successfully launched two startup projects and made great contributions to raise them as the profiting and leading companies in Turkey.

Beyond that, I like sharing my experience, mostly about Java and Spring, writing technical articles to contribute, for developing the open-source community.

As a personal hobby, I have an interest in game development and gamified concepts for web applications.

If you would like to know me better, do not hesitate to strike up a conversation with any channel below:
<p class="about-social">
  {% for social in site.social %}
    {% if social.url %}
        <a class="icon-{{ social.icon }}" href="{{ social.url }}" title="{{ social.desc }}">
          <i class="fa fa-{{ social.icon }}"></i>
        </a>
        &nbsp;&nbsp;·&nbsp;&nbsp;
    {% endif %}
  {% endfor %}
  <a class="icon-email" href="mailto:{{ site.email }}" title="Send Email">
    <i class="fa fa-envelope"></i>
  </a>
</p>
