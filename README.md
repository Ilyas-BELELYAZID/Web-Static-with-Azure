<div align="center">
<h1>Explication approfondie des étapes prévue : Hébergement d'un Site Statique sur Azure</h1>
<p>Voici une analyse détaillée de chaque étape de votre projet, avec son contexte et ses implications professionnelles.</p>
</div>

# 1. Set Up Azure Account (Créer un compte Azure)
- **Description :** C'est l'étape 0. Elle consiste à s'inscrire sur le portail Azure. Cela nécessite généralement une identité (compte Microsoft) et un moyen de paiement (même pour un compte gratuit, pour la vérification). Dans un cadre académique, vous utilisez peut-être un abonnement "Azure for Students" ou un crédit offert par votre établissement.
- **Pourquoi ?** Le compte est votre "clé d'entrée" dans l'écosystème Azure. Il est lié à un Tenant Azure Active Directory (désormais Entra ID), qui gère les identités. Ce tenant contient des **Subscriptions** (Abonnements), qui sont les unités de facturation. Vous créez des ressources (comme le stockage) à l'intérieur de ces abonnements.
- **Contexte Professionnel :** Cette étape est liée à l'**Identité et la Gouvernance (IAM)**. En entreprise, vous ne créez pas un compte ; on vous donne un utilisateur au sein du tenant de l'entreprise avec des permissions précises (RBAC - Role-Based Access Control) sur un abonnement de production ou de développement.

# 2. Create a Storage Account (GPv2) (Créer un compte de stockage)
- **Description :** C'est la première ressource que vous créez. Un compte de stockage est un service PaaS (Platform as a Service) qui fournit un espace de stockage dans le cloud. "General-purpose v2" (GPv2) est le type de compte moderne qui supporte tous les types de stockage (Blobs, Files, Queues, Tables).
- **Pourquoi ?** Vous avez besoin d'un endroit pour stocker vos fichiers (HTML, CSS, etc.). Un compte de stockage est l'offre la plus simple et la moins chère pour stocker des "Blobs" (Binary Large Objects), qui est le terme d'Azure pour "fichiers".
- **Contexte Professionnel :** Le choix de la région (ex: "France Centre", "West Europe") est critique pour la latence (proximité des utilisateurs) et la **souveraineté des données** (respect des lois, comme le RGPD). Le choix du niveau de réplication (LRS, GRS) impacte la haute disponibilité et le coût.

# 3. Enable Static Website Hosting (Activer l'hébergement de site statique)
- **Description :** C'est une simple case à cocher dans les paramètres de votre compte de stockage.
- **Pourquoi ?** C'est l'étape "magique". En activant cette option, Azure fait deux choses :
  1. Il crée un conteneur spécial nommé `$web`;
  2. Il expose un "endpoint" (une URL) public qui se comporte comme un serveur web. Il transforme ce qui n'était qu'un "disque dur dans le cloud" en un **serveur web PaaS**. Il gère automatiquement les requêtes HTTP, sert le "index.html" par défaut, gère les pages d'erreur ("404.html"), etc.
- **Contexte Professionnel :** C'est un exemple parfait d'architecture **Serverless** (Sans Serveur). Vous n'avez aucun serveur à gérer, aucun OS à patcher. Il s'adapte automatiquement à la charge (scalability) et vous ne payez que pour le stockage (Go/mois) et le trafic sortant (Go transférés).

# 4. Upload Website Files to $web (Télécharger les fichiers)
- **Description :** Vous copiez vos fichiers `index.html`, `style.css`, `script.js` et vos images dans le conteneur `$web` créé à l'étape 3. Vous pouvez le faire via le portail web, l'outil "Azure Storage Explorer", ou en ligne de commande (Azure CLI).
- **Pourquoi ?** Le conteneur `$web` est la "racine" (root directory) de votre site web. Ce que vous mettez à l'intérieur est ce que le serveur web va servir.
- **Contexte Professionnel :** En entreprise, cette étape n'est **jamais manuelle**. Elle est automatisée via un pipeline **CI/CD (Continuous Integration / Continuous Deployment)**. Un développeur "push" son code sur Git, ce qui déclenche un "workflow" (GitHub Actions, Azure Pipelines) qui construit le site (si nécessaire) et déploie (upload) les fichiers sur `$web` automatiquement.

# 5. Get the Website URL (Obtenir l'URL)
- **Description :** Une fois l'étape 3 terminée, Azure vous donne une URL unique, appelée "Endpoint principal". Elle ressemble à `https://<nom-de-votre-compte>.zXX.web.core.windows.net/`.
- **Pourquoi ?** C'est l'adresse publique de votre site. Vous pouvez la coller dans un navigateur et voir votre site fonctionner. C'est le résultat tangible de votre travail.
- **Contexte Professionnel :** Cette URL est une URL "technique". Elle est utilisée pour les tests, mais n'est jamais montrée à un client final. Elle sert d'origine pour des services plus avancés (comme un CDN).

# 6. Set Permissions (Définir les permissions)
- **Description :** Par défaut, les conteneurs de stockage sont **privés**. Pour qu'un site web fonctionne, le monde entier doit pouvoir lire vos fichiers. Cette étape consiste à régler le niveau d'accès du conteneur $web sur "Accès en lecture anonyme pour les blobs".
- **Pourquoi ?** Sans cela, chaque visiteur recevrait une erreur "403 Forbidden" (Accès refusé). Le "serveur web" a besoin de la permission de lire les fichiers pour les envoyer aux navigateurs.
- **Contexte Professionnel :** C'est un point de **sécurité critique**. L'étape 6 est en fait une "mauvaise pratique" si elle est mal faite. L'approche moderne (décrite dans l'étape 8 avec le CDN) consiste à laisser le conteneur **privé** et à utiliser un service (le CDN) qui s'authentifie auprès du stockage via une "Origin Access Identity" (OAI). Cela garantit que personne ne peut contourner le CDN et accéder directement au stockage.

# 7. Add a Custom Domain (Optionnel) (Ajouter un domaine personnalisé)
- **Description :** Cela consiste à acheter un nom de domaine (ex: `mon-projet.com`) chez un "registrar" (OVH, GoDaddy...) et à le faire pointer vers votre URL technique (étape 5) via une configuration DNS (un enregistrement CNAME).
- **Pourquoi ?** Pour le professionnalisme et la "marque". `mon-projet.com` est mémorable et crédible ; `mon-projet.z22.web.core.windows.net` ne l'est pas.
- **Contexte Professionnel :** Ce n'est pas optionnel, c'est **obligatoire** pour toute application en production. C'est aussi à cette étape que l'on gère le **HTTPS** (le cadenas SSL/TLS). L'hébergement statique seul a un support limité pour le HTTPS sur les domaines personnalisés, ce qui nous amène à l'étape finale...

# 8. Monitor and Optimize (Monitorer et Optimiser)
- **Description :** L'étape 8 mentionne deux services cruciaux : Azure Monitor et Azure CDN.
- **Pourquoi et Contexte Professionnel :**
  - **Azure Monitor :** Vous permet de savoir ce qu'il se passe. Qui visite votre site ? Y a-t-il des erreurs ? Quelle est la latence ? C'est la base de l'**Observabilité**, indispensable pour savoir si votre application fonctionne bien;
  - **Azure CDN (Content Delivery Network) :** C'est l'évolution la plus importante. Un CDN est un réseau de serveurs tout autour du monde. Au lieu de servir votre site depuis un seul endroit (ex: France), le CDN en fait des copies (un "cache") dans des serveurs à New York, Tokyo, Sydney...
    - **Bénéfice 1 (Performance) :** Un utilisateur à New York accède à votre site depuis un serveur à New York. Le site se charge quasi-instantanément;
    - **Bénéfice 2 (Sécurité & Coût) :** Le CDN gère gratuitement le **HTTPS** pour votre domaine personnalisé (étape 7). Il absorbe aussi les attaques (DDoS) et réduit vos coûts de "bande passante" (trafic sortant).

En résumé : on commence par une solution simple (Étapes 1-6) puis on la transforme en une solution professionnelle (Étapes 7-8) en ajoutant un domaine, la sécurité HTTPS et une performance globale avec un CDN.


<div align="center">
<h1>Project Specifications (Cahier de charges): Professional Cloud-Native Web Application</h1>
<p>Hosting a simple static website on an Azure Storage Account.</p>
</div>

# 1. Project Overview
**Initial Concept:** Hosting a simple static website on an Azure Storage Account.
**Professional Evolution:** To transform the basic static website into a secure, scalable, highly available, and automatically deployed cloud-native application. This project will demonstrate core competencies in modern cloud architecture, DevOps, and Infrastructure as Code (IaC), making it suitable for both a comprehensive academic module and a professional portfolio.

# 2. Core Objectives
- **Automation:** Eliminate all manual deployment steps ("click-ops");
- **Infrastructure as Code (IaC):** All cloud resources must be defined in code;
- **Security:** Implement a "secure by default" posture, moving beyond simple public read access;
- **Performance & Scalability:** Ensure the website loads quickly for global users;
- **Serverless:** Introduce dynamic functionality without managing servers.

# 3. Proposed Architecture
This architecture evolves the simple User -> Storage Account model into a professional, multi-component solution.

## 3.1. User Access & Content Delivery Architecture

```
[ User ] --> [ Azure CDN ] --> [ Azure Storage Account (Static Files) ]
                     |
                     \--> [ Azure Function App ] (for dynamic API calls, e.g., /api/contact)
```

## 3.2. Developer & Deployment (DevOps) Architecture

```
[ Developer ] --> [ Git Push (GitHub / Azure Repos) ]
                          |
                          | (Trigger)
                          v
[ CI/CD Pipeline (GitHub Actions or Azure Pipelines) ]
     |
     | (Step 1: Deploy Infrastructure)
     v
[ Infrastructure as Code (Bicep / ARM) ] --> [ Deploys/Updates Azure Resources (Storage, CDN, Function) ]
     |
     | (Step 2: Deploy Application)
     v
[ Deploys static files ] --> [ $web container in Storage Account ]
[ Deploys function code ] --> [ Azure Function App ]
```

# 4. Functional Requirements
- The system must serve the static website content (HTML, CSS, JS);
- The system must expose a dynamic API endpoint (e.g., /api/submitForm) that allows the website's JavaScript to submit data (like a contact form);
- This API endpoint will be a serverless Azure Function that processes the incoming data (e.g., logs it, sends an email).



# 5. Non-Functional Requirements (The "Pro" Requirements)

| **Category**       | **Basic Implementation**                                            | **Professional Implementation**                                                                                                                                                            |
|--------------------|---------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Deployment**     | Manual upload of files to `$web` container via Azure Portal.        | **Automated CI/CD Pipeline.** A `git push` to the `main` branch automatically builds and deploys the entire application.                                                                   |
| **Infrastructure** | Manual creation of resources (Storage Account) in the Azure Portal. | **Infrastructure as Code (IaC).** All resources (Storage, CDN, Function App) are defined in **Bicep** or **ARM Templates**. The infrastructure is version-controlled with the application. |
| **Security**       | "Set public read access for the `$web` container." (Step 6)         | **This is insecure.** Instead:<br/>1. The Storage Account will be **private**.<br/>2. The **Azure CDN** will be the only service allowed to access the storage (using Origin Access).<br/>3. All traffic will be served over **HTTPS** (Azure-managed certificate on the CDN).  |
| **Performance**    | "Monitor and Optimize: Use... Azure CDN" (Optional - Step 8)        | **Core Architecture.** The CDN is not optional. It is the primary entry point for all users, providing low-latency global caching and SSL termination.                                     |
| **Monitoring**     | "Use Azure Monitor" (Vague - Step 8)                                | **Specific Monitoring.** <br/>1. Set up **Availability Tests** in Azure Monitor to ping the website URL.<br/>2. Create **Alert Rules** for 4xx/5xx error spikes from the CDN or Function App.       |
| **Domain**         | "Add a Custom Domain (Optional)" (Step 7)                           | **Required.** The project will be configured with a custom domain (e.g., `my-project.com`), and the CDN will manage the SSL certificate for it.                                            |


# 6. Key Deliverables (For Academic Submission)
  1. **A live, publicly accessible URL** for the application;
  2. **A link to the Git repository** (e.g., GitHub);
  3. **The Infrastructure as Code (IaC) files** (e.g., .bicep files) within the repository;
  4. **The CI/CD pipeline definition file** (e.g., .github/workflows/main.yml) within the repository;
  5. **A short report (README.md)** explaining the architecture, the choices made (e.g., why Bicep?), and how to deploy the project from scratch.

# 7. Recommendations & Phased Rollout (Roadmap)
Do not try to build this all at once. Follow these steps.

- **Phase 1: The Base (Your Image)**
  - Do exactly what the image says. Get the site running manually. This is your baseline.
- **Phase 2: Infrastructure as Code (IaC)**
  - Write a Bicep or ARM template that re-creates your static site storage account (Step 2 & 3). Delete your manual one and deploy it using this script.

- **Phase 3: The Full Architecture (IaC)**
  - Expand your IaC template to add the Azure CDN and the Azure Function App.
  - Configure the CDN to point to your storage account and add your custom domain.
  - Secure the storage account (make it private) and configure CDN origin access.

- **Phase 4: Automation (CI/CD)**
  - Create a GitHub Actions (or Azure Pipelines) workflow.
  - This workflow will have two jobs:
  - deploy-infrastructure: Runs your Bicep/ARM template.
  - deploy-application: Uploads the static files to the $web container (using az storage blob upload-batch) and deploys the function code.

- **Phase 5: Dynamic Content**
  - Implement the JavaScript on your static site to call your new /api/ endpoint and test the Azure Function.

# Mini-Project Report:

```
Table of Contents

1. Introduction
  1.1. Project Problematic & Context
  1.2. Project Objectives
  1.3. Chosen Solution Overview
  1.4. Report Structure

2. Cloud Solution Architecture
  2.1. Architectural Diagram
  2.2. Core Azure Services
    2.2.1. Azure Resource Group
    2.2.2. Azure Storage Account (GPv2)
    2.2.3. Static Website Hosting Feature
  2.3. Networking and Security
    2.3.1. Public Access Endpoint
    2.3.2. Access Control (Permissions)

3. Implementation Walkthrough
  3.1. Prerequisite: Azure Account and Subscription
  3.2. Step 1: Creating the Storage Account
  3.3. Step 2: Enabling the Static Website Feature
  3.4. Step 3: Uploading Website Files (Portfolio Template)
  3.5. Step 4: Securing and Validating Access
  3.6. Final Result: The Primary Endpoint URL

4. Optimization and Professionalization (Going Further)
  4.1. Performance: Implementing Azure CDN
    4.1.1. Why a CDN is Needed
    4.1.2. Configuration Steps
  4.2. Identity: Configuring a Custom Domain
  4.3. Security: Enforcing HTTPS with the CDN

5. Monitoring and Cost Analysis
  5.1. Monitoring (Using Azure Monitor)
  5.2. Cost Analysis
    5.2.1. Estimated Monthly Cost
    5.2.2. Comparison with Traditional Hosting

6. Conclusion
  6.1. Summary of Work
  6.2. Challenges Encountered
  6.3. Future Perspectives (e.g., CI/CD Automation)

7. Appendices
  Appendix A: Key Screenshots
  Appendix B: Link to the Live Website
  Appendix C: Link to the GitHub Repository (for the portfolio code)
```
