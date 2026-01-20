---
Class: paper
created: {{importDate|format("YYYY-MM-DD")}}
title: {% if title %}{{title | replace(':', '') | replace('[', '') | replace(']', '')}}{% elseif caseName%}{{caseName | replace(':', '') | replace('[', '') | replace(']', '')}}{% elseif subject%}{{subject | replace(':', '') | replace('[', '') | replace(']', '')}}{% endif %}
cite_key: {{citationKey}}
year: {{date | format ("YYYY")}}
author: 
{%- if creators.length > 0 %}
{%- for creator in creators %} 
{%- if creator.creatorType == "author" %} 
- {% if creator.name %}"[[{{creator.name}}]]"{% else %}"[[{{creator.firstName}} {{creator.lastName}}]]"{% endif %}
{%- endif %} 
{%- endfor %}
{%- endif %}
status:
  - 👀 Queue
tags: 
link: {{url}}
aliases: 
  - {%- if shortTitle %} {{shortTitle | nl2br}} {%- endif %}
---

> [!info]+ Cite – {% for attachment in attachments | filterby("path", "endswith", ".pdf") %}[PDF](file://{{attachment.path | replace(" ", "%20")}}) [Zotero{% if not loop.first %} {{loop.index}}{% endif %}]({{attachment.desktopURI|replace("/select/", "/open-pdf/")}}){% if not loop.last %}, {% endif %}{% endfor %}
> Github: 
> Youtube: 
> {{bibliography}}

> [!tldr]+ Abstract
> {{abstractNote | nl2br}}

---
### Summary


{% if markdownNotes %}
### Note
{{markdownNotes}}{% endif %}

---

{#- WARNING: Everything below this line will get overwritten on re-import! -#}
{#- COLOR VARIABLES -#}
{%-
    set zoteroColors = {
        "#2ea8e5": "blue",
        "#5fb236": "green",
        "#a28ae5": "purple",
        "#ffd400": "yellow",
        "#ff6666": "red",
        "#f19837": "orange",
        "#e56eee": "magenta",
        "#aaaaaa": "grey"
    }
-%}
{%-
   set colorHeading = {
        "yellow": "✅ Comments",
        "red": "❓Questions",
        "green": "📚 Further Reading",
        "blue": "🧩 Concepts",
        "other": "Misc"
   }
-%}
{#- ANNOTATION TYPES -#}
{%- macro calloutHeader(type) -%}
    {%- switch type -%}
        {%- case "highlight" -%}
        Highlight
        {%- case "image" -%}
        Image
        {%- default -%}
        Note
    {%- endswitch -%}
{%- endmacro %}
{#- SET CUSTOM ANNOTATION COLORS -#}
{%- set newAnnot = [] -%}
{%- set newAnnotations = [] -%}
{%- set annotations = annotations | filterby("date", "dateafter", lastImportDate) %}
{% if annotations.length > 0 %}
{%- for annot in annotations -%}
    {%- if annot.color in zoteroColors -%}
        {%- set customColor = zoteroColors[annot.color] -%}
    {%- elif annot.colorCategory|lower in colorHeading -%}
    	{%- set customColor = annot.colorCategory|lower -%}
    {%- else -%}
	    {%- set customColor = "other" -%}
    {%- endif -%}
    {%- set newAnnotations = (newAnnotations.push({"annotation": annot, "customColor": customColor}), newAnnotations) -%}
{%- endfor -%}

{#- INSERT ANNOTATIONS -#}
{#- Loops through each of the available colors and only inserts matching annotations -#}
{#- This is a workaround for inserting categories in a predefined order (instead of using groupby & the order in which they appear in the PDF) -#}

{%- for color, heading in colorHeading -%}
{%- for entry in newAnnotations | filterby ("customColor", "startswith", color) -%}
{%- set annot = entry.annotation -%}

{%- if entry and loop.first %}

### {{colorHeading[color]}}
{%- endif %}

> [!quote{{"|" + color if color != "other"}}]+ {{calloutHeader(annot.type)}} ([p. {{annot.pageLabel}}](zotero://open-pdf/library/items/{{annot.attachment.itemKey}}?page={{annot.pageLabel}}&annotation={{annot.id}}))

{%- if annot.annotatedText %}
> {% if annot.hashTags %}[[{{annot.hashTags|replace("#", "")}}]]: {% endif -%}
{{annot.annotatedText|nl2br}}
{%- endif %}

{%- if annot.imageRelativePath %}
> ![[{{annot.imageRelativePath}}]]
{%- endif %}

{%- if annot.ocrText %}
> {{annot.ocrText}}
{%- endif %}

{%- if annot.comment %}
> → **{{annot.comment|nl2br}}**
{%- endif -%}

{%- endfor -%}
{%- endfor -%}
{% endif %}