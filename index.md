---
title: Instruções online hospedadas
permalink: index.html
layout: home
---

Este repositório contém os exercícios práticos de laboratório para o curso da Microsoft [DP-420 Projetar e implementar aplicativos nativos de nuvem usando o Microsoft Azure Cosmos DB][course-description] e os [módulos individuais no Microsoft Learn][learn-collection] equivalentes. Os laboratórios foram pensados para acompanhar os materiais de aprendizado e para você praticar o uso das tecnologias descritas.

> &#128221; Para concluir estes exercícios, você precisará de uma assinatura do Microsoft Azure. Inscreva-se para uma avaliação gratuita em [https://azure.microsoft.com][azure].

# Laboratórios

## Laboratórios C#

{% assign labs = site.pages | where_exp:"page", "page.url contains '/instructions'" %} {% assign csharp_setup_labs = "" | split: "," %} {% assign csharp_regular_labs = "" | split: "," %} {% assign genai_setup_labs = "" | split: "," %} {% assign genai_python_labs = "" | split: "," %} {% assign genai_javascript_labs = "" | split: "," %}

{% for activity in labs %} {% assign segments = activity.url | split: "/" %}

  {% if segments[1] == "instructions" and segments.size == 3 %} {% if activity.lab.module contains "Setup" %} {% assign csharp_setup_labs = csharp_setup_labs | push: activity %} {% else %} {% assign csharp_regular_labs = csharp_regular_labs | push: activity %} {% endif %}
  
  {% elsif activity.url contains '/gen-ai/python/instructions' %} {% assign genai_python_labs = genai_python_labs | push: activity %}
  
  {% elsif activity.url contains '/gen-ai/javascript/instructions' %} {% assign genai_javascript_labs = genai_javascript_labs | push: activity %}
  
  {% elsif activity.url contains '/gen-ai/common/instructions' and activity.lab.module contains "Setup" %} {% assign genai_setup_labs = genai_setup_labs | push: activity %} {% endif %} {% endfor %}

---

### **Laboratórios de configuração**

| Módulo | Laboratório |
| --- | --- |
{% for activity in csharp_setup_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

### **Laboratórios**

| Módulo | Laboratório |
| --- | --- |
{% for activity in csharp_regular_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

## **Laboratórios de IA generativa**

### **Laboratórios de configuração comuns**

| Módulo | Laboratório |
| --- | --- |
{% for activity in genai_setup_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

### **Laboratórios JavaScript**

| Módulo | Laboratório |
| --- | --- |
{% for activity in genai_javascript_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

### **Laboratórios Python**

| Módulo | Laboratório |
| --- | --- |
{% for activity in genai_python_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

[azure]: https://azure.microsoft.com
[course-description]: https://docs.microsoft.com/learn/certifications/courses/dp-420t00
[learn-collection]: https://docs.microsoft.com/users/msftofficialcurriculum-4292/collections/1k8wcz8zooj2nx
