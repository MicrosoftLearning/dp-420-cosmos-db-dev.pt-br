---
title: Instruções online hospedadas
permalink: index.html
layout: home
---

Este repositório contém os exercícios práticos de laboratório para o curso da Microsoft [DP-420 Projetar e implementar aplicativos nativos de nuvem usando o Microsoft Azure Cosmos DB][course-description] e os [módulos individuais no Microsoft Learn][learn-collection] equivalentes. Os laboratórios foram pensados para acompanhar os materiais de aprendizado e para você praticar o uso das tecnologias descritas.

> &#128221; Para concluir estes exercícios, você precisará de uma assinatura do Microsoft Azure. Inscreva-se para uma avaliação gratuita em [https://azure.microsoft.com][azure].

## Laboratórios

{% assign labs = site.pages | where_exp:"page", "page.url contains '/instructions'" %}
| Módulo | Laboratório |
| --- | --- |
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

[azure]: https://azure.microsoft.com
[course-description]: https://docs.microsoft.com/learn/certifications/courses/dp-420t00
[learn-collection]: https://docs.microsoft.com/users/msftofficialcurriculum-4292/collections/1k8wcz8zooj2nx
