---
title: 온라인 호스팅 지침
permalink: index.html
layout: home
ms.openlocfilehash: c612655617c9fb1f1baef2b3a78d0cb87f0e3d18
ms.sourcegitcommit: 9a51ea796ef3806ab9e7ec1ff75034b2f929ed2a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/28/2022
ms.locfileid: "137907161"
---
# <a name="content-directory"></a>콘텐츠 디렉터리

다음은 각 랩 연습 및 데모의 하이퍼링크입니다.

## <a name="labs"></a>랩

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| 모듈 | 랩 |
| --- | --- | 
{% for activity in labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

