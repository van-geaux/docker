# System

|    |    | Positives | Negatives |
|----|----|----|----|
| [Watchtower](/docs/system/watchtower.md)**⭐** | Watchtower is an application that monitors your running containers for changes to the image. When you push an update to your Docker registry, Watchtower detects the changes and pulls down the new copy of the image. It will then restart the container using the new image and the same start options of the original. | Makes upgrading apps easy | Can break things if not controlled |

# Critical

|    |    | Positives | Negatives |
|----|----|----|----|
| [Yacht](/docs/system/yacht.md) | A web interface for managing docker containers with an emphasis on templating to provide one-click deployments of dockerized applications. Think of it like a decentralized app store for servers that anyone can make packages for. |    |    |

# Database

|    |    | Positives | Negatives |
|----|----|----|----|
| [MariaDB](/docs/system/mariadb.md) **⭐** | MariaDB is a community-developed, commercially supported fork of the MySQL relational database management system, intended to remain free and open-source software under the GNU General Public License. |    |    |
| [MySQL](/docs/system/mysql.md) **⭐** | MySQL is an open-source relational database management system. Its name is a combination of "My", the name of co-founder Michael Widenius's daughter My, and "SQL", the acronym for Structured Query Language. |    |    |
| [PostgreSQL](/docs/system/postgresql.md) **⭐** | PostgreSQL, also known as Postgres, is a free and open-source relational database management system emphasizing extensibility and SQL compliance. It was originally named POSTGRES, referring to its origins as a successor to the Ingres database developed at the University of California, Berkeley. |    |    |

# Network

|    |    | Positives | Negatives |
|----|----|----|----|
| [NGINX Proxy Manager](/docs/system/nginx-proxy-manager.md) | The Nginx Proxy Manager offers **a** convenient tool for managing proxy hosting. The proxy manager makes it relatively easy to forward traffic to your services, whether running on the cloud or your home network. This tutorial introduces the Nginx Proxy Manager and illustrates how to start using it. | Easy to use | Becomes harder to maintain once you have a lot to forward |
| [Traefik](/docs/system/traefik.md) **⭐** | Traefik is a leading modern reverse proxy and load balancer that makes deploying microservices easy. Traefik integrates with your existing infrastructure components and configures itself automatically and dynamically. | Easy to maintain | Somewhat hard to setup the first time |

# Security

|    |    | Positives | Negatives |
|----|----|----|----|
| [Authelia](/docs/system/authelia.md) **⭐** | Authelia is an open-source authentication and authorization server and portal fulfilling the identity and access management (IAM) role of information security in providing multi-factor authentication and single sign-on (SSO) for your applications via a web portal. | Provides 2FA and OIDC | Hard to deploy the first time, high learning curve |
| [Vaultwarden](/docs/system/vaultwarden.md) **⭐** | Vaultwarden is an open-source password manager that is designed to be self-hosted. Although Vaultwarden is not technically a fork of Bitwarden, it is instead a recreation of the backend using Rust. | Easier to install and less heavy to the system compared to Bitwarden | Not an official Bitwarden, use with caution |

# Backup

|    |    | Positives | Negatives |
|----|----|----|----|
| [Duplicati](/docs/system/duplicati.md) | Duplicati is a backup client that securely stores encrypted, incremental, compressed remote backups of local files on cloud storage services and remote file servers. |    |    |

# System support

|    |    | Positives | Negatives |
|----|----|----|----|
| [phpMyAdmin](/docs/system/phpmyadmin.md) **⭐** | phpMyAdmin is a free and open source administration tool for MySQL and MariaDB. As a portable web application written primarily in PHP, it has become one of the most popular MySQL administration tools, especially for web hosting services. |    |    |
| [Sshwifty](/docs/system/sshwifty.md) :star: | Sshwifty is a web-based SSH and Telnet client that allows you to access these services directly from any standard web browser. It can be deployed on your computer or server, providing a convenient and accessible interface for SSH and Telnet connections. | Easy to use | There is a server-client time-sync protection  |

# Other