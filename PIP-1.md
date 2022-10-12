# PIP-1 Point-Chain common components

## Motivation

Point Chain intends to have several ecosystem layer applications. Those applications can reuse similar components. Reusing these components can reduce the amount of work and make applications better connected with each other.


## Details

There are 5 types of reusable components on Point Network

* Development tools - It reduces need to repeate same routine tasks developing each application. same time having reusable components, we can have common tools for its testing, debugging, configuration etc. Example: testing framework for our apps which ease deployment and configuration of our smart contracts for development needs.
* Smart contracts - Code re-usability allows our developers to increase the quality of their code, in terms of security, resource efficiency and readability. Example: Upgradable Proxy with integration of our identity can be shared between all aps which use Identity contract.
* Reusable backend - Example: background tasks, like message processing can be shared between apps.
* Web UI - A great percentage of dApps will share the same components, such as comments or chat, and their UI implementations can be reused. Example: UI 'reactions` components can be shared between several Social apss in the ecosystem.
* Data - Similarly to how UIs can be shared among different dApps, the data of these components can be reused. Example: two or more dApps can share the same chat conversations, along with the contacts associated to these.

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