---
date: "2023-08-08"
sections:
- block: about.biography
  content:
    title: Biography
    username: admin
  id: about
- block: portfolio
  content:
    filters:
      folders:
      - project
    title: Projects
    summary: "Welcome to my portfolio, a curated collection of projects that I have taken immense pleasure in crafting. These endeavors represent a compilation of concise undertakings, which not only afforded me personal gratification but also facilitated the acquisition of novel skills. Within this assortment, you will encounter animated graphs, bespoke cartography, and sophisticated statistical techniques. It is noteworthy that several of these initiatives originated during interactive coding sessions conducted within my instructional capacity for the Statistical Programming / Data Analysis in R course at ELTE. Additionally, certain endeavors were cultivated through collaborative efforts within the professional sphere, including projects centered around replication analysis."
  design:
    columns: "1"
    flip_alt_rows: false
    view: compact
  id: projects
- block: portfolio
  content:
    buttons:
    - name: All
      tag: '*'
    - name: Methods
      tag: methods
    - name: R
      tag: R
    default_button_index: 0
    filters:
      folders:
      - workshop
    title: Workshops
  design:
    columns: "1"
    flip_alt_rows: false
    view: showcase
  id: workshops
- block: collection
  content:
    filters:
      featured_only: true
      folders:
      - publication
    title: Featured Publications
  design:
    columns: "2"
    view: card
  id: featured
- block: collection
  content:
    filters:
      exclude_featured: true
      folders:
      - publication
    text: |-
      {{% callout note %}}
      Quickly discover relevant content by [filtering publications](./publication/).
      {{% /callout %}}
    title: Recent Publications
  design:
    columns: "2"
    view: citation
- block: tag_cloud
  content:
    title: Popular Topics
  design:
    columns: "2"
- block: contact
  content:
    address:
      city: Budapest
      country: Hungary
      country_code: hu
      postcode: "1064"
      street: Izabella u. 46.
    appointment_url: http://nagytamas-meeting.youcanbook.me
    autolink: true
    contact_links:
    - icon: twitter
      icon_pack: fab
      link: https://twitter.com/nagyt
      name: DM Me
    directions: Take the elevator to the 5th floor, follow the corridor to Room 521
    email: nagy.tamas@ppk.elte.hu
    office_hours:
    - Monday 10:00 to 13:00
    - Wednesday 09:00 to 10:00
    subtitle: null
    title: Contact
  design:
    columns: "2"
  id: contact
title: null
type: landing
---
