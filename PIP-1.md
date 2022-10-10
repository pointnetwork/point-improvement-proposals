# PIP-1 Point-Chain common components

## Motivation

Point chain intended to have several ecosystem layer applications. Those application can reuse similar components. It can reduce amount of work and make applications better connected with each other.


## Details

There are 5 types of reusable components on Point network

* Development tools 
* Smart contracts
* Reusable backend
* Web UI 
* Data

### Reusable Smart Contracts

#### Identity related

* Reusable indentity integration with the Proxy pattern

#### Reusable Social components

Point network intended to have several Social applications including

* Point.Social
* Point.Blog
* Point.Mail
* Point.Video

These applications contains many components which duplicates functionality. The real difference of these applications is UX focus. Some of applications are 
* Focused on public messaging like Point.Social which intended to be a WEB3 twitter,  
* Focused on private messaging like Point.Mail
* Focused on video like Point.Video which intended to be a reincarnation of WEB# YouTube

Same time all of these applications have similar functionality on the UI level, like 
* Point.Social needs private messaging and Point.Mail contract can be reused for this purpose. 
* Point.Video needs comments and can share this functionality from Point.Social and Point.Blog.
* Point.Social & Point.Blog can have same smart contracts being differentiated only on the UI level. So Point. Social will be focused on communication about short posts while Point.Blog UI can be designed for longread posts.

Such approach requires modular features architecture for Point applications giving infinit ability to reuse components while being managed with same identities.