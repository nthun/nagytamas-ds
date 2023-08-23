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
    text: "The following quick projects are not directly connected to my research, but were fun to create and also taught me some cool new stuff like animated graphs, custom maps, and nifty statistical methods. Some of these projects sprouted from live coding sessions I hosted in my 'Statistical Programming / Data Analysis in R' course at ELTE. Another few came from professional collaborations like replication projects. "
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
    text: "I regularly hold workshops for researchers about advanced research methodology and R programming.  Please do not hesitate to contact me if you want to invite me to hold a workshop!"
  design:
    columns: "1"
    flip_alt_rows: false
    view: showcase
  id: workshops
- block: collection
  content:
    filters:
      exclude_featured: false
      folders:
      - publication
    text: |-
      {{% callout note %}}
      Quickly discover relevant content by [filtering publications](./publication/).
      {{% /callout %}}
    title: Recent Publications
  design:
    columns: "2"
    view: list
  id: publications
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
    email: nagy.tamas@ppk.elte.hu
    subtitle: null
    title: Contact
  design:
    columns: "2"
  id: contact
title: null
type: landing
---
