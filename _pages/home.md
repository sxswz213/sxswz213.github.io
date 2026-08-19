---
title: "Home"
layout: homelay
sitemap: false
permalink: /
---

<br/>
<style>
.home-intro {
  color: #444;
}

.home-intro a {
  color: #3f9f86;
}

.home-intro h3 {
  color: #000;
}
</style>

<div class="home-intro" markdown="1">

### Hello👋, I'm Zhiyao!

🌱I am currently pursuing a Ph.D. in Artificial Intelligence at Northwestern Polytechnical University (NWPU), 
advised by <a href="https://scholar.google.com.hk/citations?user=vuGkcaoAAAAJ&amp;hl=zh-CN" target="_blank" rel="noopener noreferrer">Prof. Zhen Wang</a> and <a href="https://shuyuehu.github.io/index.html" target="_blank" rel="noopener noreferrer">Dr. Shuyue Hu</a>.
My research focuses on **multi-agent systems** driven by large language models (LLMs) and vision-language models (VLMs).
I completed my undergraduate degree in Information Security at Northwestern Polytechnical University(NWPU) in June 2024. 

💡I'm particularly interested in developing robust and practical AI systems capable of effectively managing diverse and dynamic scenarios. This involves enhancing the adaptability, efficiency, and reliability of AI-driven solutions to tackle complex real-world problems.

👯Beyond academics, I hold a black belt in Taekwondo and have achieved the highest level in electronic keyboard. Whether in research or personal life, I enjoy challenges and believe in bringing curiosity and enthusiasm to everything I do.

📫Feel free to connect—I’m always excited to discuss innovative ideas or collaborate on cutting-edge research!

</div>

<div class="container">
<div class="row">
<center>
<img src="{{ site.url }}{{ site.baseurl }}/images/3.jpeg" width="100%" style="opacity: 0.8;"/><br/>
  <br/>
</center>
</div>
</div>
<br/>

---

### [Publications](/publications/)

{% bibliography --query @*[keywords ^= main] %}

#### Other Works

{% bibliography --query @*[keywords != main] --max 1 %}

[See all publications →](/publications/)
{: .publication-more-link}

---

### [Academic Services & Activities](/services/)

<div class="home-services">
{% for group in site.data.academic_services %}
<h4>{{ group.section }}</h4>
<ul>
  {% for item in group.items %}
    <li>{{ item }}</li>
  {% endfor %}
</ul>
{% endfor %}
</div>
