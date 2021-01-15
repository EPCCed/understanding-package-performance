---
layout: lesson
root: .  # Is the only page that doesn't follow the pattern /:path/index.html
permalink: index.html  # Is the only page that doesn't follow the pattern /:path/index.html
venue: Online
address: 
country: "UK"
language: "English"
latlng: 
humandate: 
humantime: 
startdate: 
enddate: 
instructor: 
helper: 
email: ["support@archer2.ac.uk"]
collaborative_notes: 
---
As parallel packages for computational science become more sophisticated, it becomes more difficult
for a researcher to understand the most important factors that determine end-to-end productivity
from initial input data to final result. Aspects such as file IO and data transfer can be just as
important in practice as the performance and parallel scalability of the application itself. This
course will provide an introduction to understanding your research workflow, the place of HPC
application performance within the workflow, an introduction to benchmarking parallel applications
and how you can use benchmark data to make decisions on running your research on HPC systems.

The lesson aims to answer the following questions:
  - How can I understand the end-to-end performance of my research workflow and, particularly, how 
    does my use of HPC fit into this workflow?
  - How do I measure parallel application performance and which metrics should I use and when?
  - What decisions on my use of HPC can I make based on performance measurements?

<h2 id="general">General Information</h2>

{% comment %}
  LOCATION

  This block displays the address and links to maps showing directions
  if the latitude and longitude of the workshop have been set.  You
  can use https://itouchmap.com/latlong.html to find the lat/long of an
  address.
{% endcomment %}
<p id="where">
  <strong>Where:</strong>
  This course will be taught online via Blackboard Collaborate. All attendees will
  be sent the joining link prior to the event.
</p>

{% comment %}
  DATE

  This block displays the date and links to Google Calendar.
{% endcomment %}
{% if page.humandate %}
<p id="when">
  <strong>When:</strong>
  {{page.humandate}}.
  {% include workshop_calendar.html %}
</p>
{% endif %}

{% comment %}
  SPECIAL REQUIREMENTS

  Modify the block below if there are any special requirements.
{% endcomment %}
<p id="requirements">
  <strong>Requirements:</strong> Participants must bring a laptop with a
  Mac, Linux, or Windows operating system (not a tablet, Chromebook, etc.) that they have administrative privileges
  on. They should have a few specific software packages installed (listed
  <a href="#setup">below</a>). They are also required to abide by the <a href="https://www.archer2.ac.uk/training/code-of-conduct/">ARCHER2 Training Code of Conduct</a>.
</p>

{% comment %}
  ACCESSIBILITY

  Modify the block below if there are any barriers to accessibility or
  special instructions.
{% endcomment %}
<p id="accessibility">
  <strong>Accessibility:</strong> We are committed to making this workshop
  accessible to everybody.
</p>
<p>
  Materials will be provided in advance of the workshop. If we can help making learning easier for
  you please get in touch (using contact details below) and we will attempt to provide them.
</p>

{% comment %}
  CONTACT EMAIL ADDRESS

  Display the contact email address set in the configuration file.
{% endcomment %}
<p id="contact">
  <strong>Contact</strong>:
  Please email
  {% if page.email %}
    {% for email in page.email %}
      {% if forloop.last and page.email.size > 1 %}
        or
      {% else %}
        {% unless forloop.first %}
        ,
        {% endunless %}
      {% endif %}
      <a href='mailto:{{email}}'>{{email}}</a>
    {% endfor %}
  {% else %}
    to-be-announced
  {% endif %}
  for more information.
</p>

<hr/>

> ## Prerequisites
> You should have used remote HPC facilities before. In particular, you should be happy with connecting
> using SSH, know what a batch scheduling system is and be familiar with using the Linux command line.
> You should also be happy editing plain text files in a remote terminal (or, alternatively, editing them
> on your local system and copying them to the remote HPC system using `scp`).
{: .prereq}

> ## Requirements
> Participants must bring a laptop with a Mac, Linux, or Windows operating system (not a tablet,
> Chromebook, etc.) that they have administrative privileges on. They should have a few specific software
> packages installed (listed in the Setup section below). They are also required to abide by the
> [ARCHER2 Training Code of Conduct](https://www.archer2.ac.uk/training/code-of-conduct/).
{: .prereq}

Note that this lesson uses [The Carpentries](https://carpentries.org) template but this is not a 
Carpentries lesson.

{% include links.md %}

{% comment %}

<!--  LocalWords:  prereq links.md endcomment
 -->
{% endcomment %}
