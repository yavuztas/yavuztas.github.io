---
layout: about
title: About
permalink: /about/
image: "/assets/article_images/about/about.jpg"
image2: "/assets/article_images/about/about-mobile.jpg"
---
>The problem does not lie in technology but in design. I am a true believer in code quality and I believe clear code and correct design with enough time bring absolute quality on software development.

Let me tell you about myself...

In short, I am a passionate software developer who loves **Java** technologies.

For more details about me, I am a hard-working, innovative software developer with active field experience since 2005, mostly specialized in back-end enterprise technologies using Java and Java EE. Have practical knowledge about solving problems using various design patterns and OOP techniques. I am highly skilled at Spring Framework, Spring Security, ORM frameworks especially JPA, Hibernate and a variety of backend related technologies as well as experienced about UI/UX technologies both server-side rendered like JSF, Vaadin and client-side rendered like Javascript, Jquery and frontend MVC frameworks like Angular, Aurelia. Also experienced and strictly tied with test-driven development, issue tracking, continuous integration, and source code management.

Among my recent achievements, I have successfully launched two startup projects and made great contributions to raise them as the profiting and leading company in Turkey. Beyond that, as a personal hobby, I have a great passion for game development and fill my spare time developing small but enjoyable games.

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
