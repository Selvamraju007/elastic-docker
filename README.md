# Introduction
Elastic Stack (**ELK**) Docker Composition, preconfigured with **Security**, **Monitoring**, and **Tools**; Up with a Single Command.

Stack Version: [8.8.0](https://www.elastic.co/blog/whats-new-elastic-8-8-0) 🎉  - Based on [Official Elastic Docker Images](https://www.docker.elastic.co/)
> You can change Elastic Stack version by setting `ELK_VERSION` in `.env` file and rebuild your images. Any version >= 8.0.0 is compatible with this template.

### Main Features 📜

- Configured as a Production Single Node Cluster. (With a multi-node cluster option for experimenting).
- Security Enabled By Default.
- Configured to Enable:
  - Logging & Metrics Ingestion
  - APM
  - Alerting
  - Machine Learning
  - SIEM
  - Use Docker-Compose and `.env` to configure your entire stack parameters.
- Persist Elasticsearch's Keystore and SSL Certifications.
- Self-Monitoring Metrics Enabled.
- Prometheus Exporters for Stack Metrics.
- Collect Docker Host Logs to ELK via `make collect-docker-logs`.
- Embedded Container Healthchecks for Stack Images.

#### More points

<details><summary>Expand...</summary>
<p>


- Security enabled by default using Basic license, not Trial.

- Persisting data by default in a volume.

- Run in Production Mode (by enabling SSL on Transport Layer, and add initial master node settings).

- Persisting Generated Keystore, and create an extendable script that makes it easier to recreate it every-time the container is created.

- Parameterize credentials in .env instead of hardcoding `elastich:changeme` in every component config.

- Parameterize all other Config like Heap Size.

- Add recommended environment configurations as Ulimits and Swap disable to the docker-compose.

- Make it ready to be extended into a multinode cluster.

- Configuring the Self-Monitoring and the Filebeat agent that ship ELK logs to ELK itself. (as a step to shipping it to a monitoring cluster in the future).

- The Makefile that simplifies everything into some simple commands.

</p>
</details>

-----

# Requirements

- [Docker 20.05 or higher](https://docs.docker.com/install/)
- [Docker-Compose 1.29 or higher](https://docs.docker.com/compose/install/)
- 4GB RAM (For Windows and MacOS make sure Docker's VM has more than 4GB+ memory.)

# Setup

1. Clone the Repository
     ```bash
     git clone https://github.com/Selvamraju007/elastic-docker.git
     ```
2. Initialize Elasticsearch Keystore and TLS Self-Signed Certificates
    ```bash
    $ make setup
    ```
    > **For Linux's docker hosts only**. By default virtual memory [is not enough](https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html) so run the next command as root `sysctl -w vm.max_map_count=262144`
3. Start Elastic Stack
    ```bash
    $ make elk           <OR>         $ docker-compose up -d		<OR>		$ docker compose up -d
    ```
4. Visit Kibana at [https://localhost:5601](https://localhost:5601) or `https://<your_public_ip>:5601`

    Default Username: `elastic`, Password: `changeme`

    > - Notice that Kibana is configured to use HTTPS, so you'll need to write `https://` before `localhost:5601` in the browser.
    > - Modify `.env` file for your needs, most importantly `ELASTIC_PASSWORD` that setup your superuser `elastic`'s password, `ELASTICSEARCH_HEAP` & `LOGSTASH_HEAP` for Elasticsearch & Logstash Heap Size.
    
> Whatever your Host (e.g AWS EC2, Azure, DigitalOcean, or on-premise server), once you expose your host to the network, ELK component will be accessible on their respective ports. Since the enabled TLS uses a self-signed certificate, it is recommended to SSL-Terminate public traffic using your signed certificates. 

> 🏃🏻‍♂️ To start ingesting logs, you can start by running `make collect-docker-logs` which will collect your host's container logs.

## Additional Commands

<details><summary>Expand</summary>
<p>

#### To Start Monitoring and Prometheus Exporters
```shell
$ make monitoring
```
#### To Start Tools
```shell
$ make tools
```
#### To Ship Docker Container Logs to ELK 
```shell
$ make collect-docker-logs
```
#### To Start **Elastic Stack, Tools and Monitoring**
```
$ make all
```
#### To Start 2 Extra Elasticsearch nodes (recommended for experimenting only)
```shell
$ make nodes
```
#### To Rebuild Images
```shell
$ make build
```
#### Bring down the stack.
```shell
$ make down
```

#### Reset everything, Remove all containers, and delete **DATA**!
```shell
$ make prune
```

</p>
</details>

# Configuration

* Some Configuration are parameterized in the `.env` file.
  * `ELASTIC_PASSWORD`, user `elastic`'s password (default: `changeme` _pls_).
  * `ELK_VERSION` Elastic Stack Version (default: `8.8.0`)
  * `ELASTICSEARCH_HEAP`, how much Elasticsearch allocate from memory (default: 1GB -good for development only-)
  * `LOGSTASH_HEAP`, how much Logstash allocate from memory.
  * Other configurations which their such as cluster name, and node name, etc.
* Elasticsearch Configuration in `elasticsearch.yml` at `./elasticsearch/config`.
* Logstash Configuration in `logstash.yml` at `./logstash/config/logstash.yml`.
* Logstash Pipeline in `main.conf` at `./logstash/pipeline/main.conf`.
* Kibana Configuration in `kibana.yml` at `./kibana/config`.
* Rubban Configuration using Docker-Compose passed Environment Variables.

### Setting Up Keystore

You can extend the Keystore generation script by adding keys to `./setup/keystore.sh` script. (e.g Add S3 Snapshot Repository Credentials)

To Re-generate Keystore:
```
make keystore
```

### Notes


- ⚠️ Elasticsearch HTTP layer is using SSL, thus mean you need to configure your elasticsearch clients with the `CA` in `secrets/certs/ca/ca.crt`, or configure client to ignore SSL Certificate Verification (e.g `--insecure` in `curl`).

- Adding Two Extra Nodes to the cluster will make the cluster depending on them and won't start without them again.

- Makefile is a wrapper around `Docker-Compose` commands, use `make help` to know every command.

- Elasticsearch will save its data to a volume named `elasticsearch-data`

- Elasticsearch Keystore (that contains passwords and credentials) and SSL Certificate are generated in the `./secrets` directory by the setup command.

- Make sure to run `make setup` if you changed `ELASTIC_PASSWORD` and to restart the stack afterwards.

- For Linux Users it's recommended to set the following configuration (run as `root`)
    ```
    sysctl -w vm.max_map_count=262144
    ```
    By default, Virtual Memory [is not enough](https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html).


