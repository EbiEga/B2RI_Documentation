# Summary

The Beacon v2 Reference Implementation is a software that must be installed **locally** in a Linux server/workstation. 

The software consist of [several components](https://b2ri-documentation.readthedocs.io/en/latest/beacon-v2-reference-implementation) that are distributed in two different GitHub repositories, one for [data ingestion tools](https://github.com/EGA-archive/beacon2-ri-tools) and another for the [API](https://github.com/EGA-archive/beacon2-ri-api). The software also needs external dependencies to work. In this regard, we provide several alternatives for download and installation.

## Containerized (Recommended)

### Method 1: Docker Hub

A set of Docker images can be obtained from [Docker Hub](https://hub.docker.com/r/beacon2ri/beacon_reference_implementation). Please see the documentation there.

### Method 2: Independent containers

!!! Note "Tip"
    The data ingestion tools can function without the API, so you can download one without the other.

#### Data ingestion tools (beacon2-ri-tools)

Please follow the instructions provided in the [README](https://github.com/EGA-archive/beacon2-ri-tools/blob/main/README.md#containerized).

#### Beacon v2 REST API (beacon2-ri-api)

Please follow the instructions provided in the [README](https://github.com/EGA-archive/beacon2-ri-api/blob/master/deploy/README.md).

## Non containerized (data ingestion tools)

!!! Warning "Message"
    This alternative requires a few more steps than the containerized one, but it gives the deployer more control over the tools.

We will download the **data ingestion tools** from [this](https://github.com/EGA-archive/beacon2-ri-tools) repository. Please follow the instructions provided in the [README](https://github.com/EGA-archive/beacon2-ri-tools#non-containerized).
 
The data ingestion tools need **external software** to function:

* **BCFtools** (version 1.15.1)
* **SnpEff** + databases (version 5.0)
* **MongoDB**

!!! Important
    Even if you have the **external tools** already in your system, for the sake of consistency of versions, we recommend downloading them from our servers.

We will download _BCFtools_, _SnpEff_ and _MongoDB_ utilities from a public `ftp` server (`ftp://xfer13.crg.eu`) located at CRG. We will use `wget` to get the five parts (~65G total). Each part should take around 20 min to download:

    wget ftp://FTPuser:FTPusersPassword@xfer13.crg.eu:221/beacon2_data.md5
    wget ftp://FTPuser:FTPusersPassword@xfer13.crg.eu:221/beacon2_data.part1
    wget ftp://FTPuser:FTPusersPassword@xfer13.crg.eu:221/beacon2_data.part2
    wget ftp://FTPuser:FTPusersPassword@xfer13.crg.eu:221/beacon2_data.part3
    wget ftp://FTPuser:FTPusersPassword@xfer13.crg.eu:221/beacon2_data.part4
    wget ftp://FTPuser:FTPusersPassword@xfer13.crg.eu:221/beacon2_data.part5

Once you have downloaded the 5 parts and the checksum file (\*.md5) please check that they are complete:

    md5sum beacon2_data.part? > my_beacon2_data.md5
    diff my_beacon2_data.md5 beacon2_data.md5

Now join the 5 parts by typing (note that momentarily we'll be using ~128G):

    cat beacon2_data.part? > beacon2_data.tar.gz 
    rm beacon2_data.part?

OK, everything ready to untar the file:

    tar -xvf beacon2_data.tar.gz
    cd snpeff/v5.0 ; ln -s GRCh38.99 hg38 # In case the symbolic link does not exist already

*NB*: Feel free now to erase ```beacon2_data.tar.gz``` if needed.

Great! now we recommend moving the directories to your favourite location and keep the path. We will be using the paths to **set up** some variables for **SnpEff** and for **beacon** configuration files. 

!!! Warning "About `java`"
    SnpEff runs with `java` which you may need to install separately. See how [here](https://ubuntu.com/tutorials/install-jre).


For **SnpEff**:

Use the path where you have left the databases and use it to change ```data.dir``` variable in ```snpEff.config``` file (located in SnpEff installation folder).
For instance, in my case:

```
#data.dir = ./data/

data.dir = /media/mrueda/4TB/Databases/snpeff/v5.0/
```

And finally....

For **Beacon**:

Open the file ```config.yaml``` (inside ```beacon_X.X.X``` dir installation) and change the paths to the files/exes according to your new locations. Note that `SnpSift` is part of `SnpEff` main distribution.

That's it!


### MongoDB installation

We're going to install it by using [docker-compose](https://docs.docker.com/compose). You need to have `docker` and `docker-compose` installed. `docker-compose` enables defining and running multi-container Docker applications.

!!! Danger "About Docker"
    It's out of the scope of this documentation to explain how to install `docker` engine and `docker-compose`.
    Please take a look to Docker [documentation](https://docs.docker.com/engine/install) if you need help with the installation.

First you need to create a file named ```docker-compose.yml``` with these contents:

```
version: '3.1'

# networks:
#   beacon-priv:
#   idp-priv:
#   pub:

services:

  ###########################################
  # MongoDB Database
  ###########################################

  mongo:
    image: mongo
    hostname: mongo
    ports:
      - 27017:27017
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
    # networks:
    #   - beacon-priv

  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: example
      ME_CONFIG_MONGODB_URL: mongodb://root:example@mongo:27017/
    # networks:
    #   - beacon-priv
```

And then run:

    docker-compose up -d

Once complete you should have two Docker processes: ```mongo``` and ```mongo-express```. You can check this by typing:

    docker ps -a

!!! Warning "About MongoDB Community version"
    We're deliberately installing `mongodb` package provided by Ubuntu, which is **not** maintained by MongoDB Inc. The _official_ distribution of MongoDB Community is called `mongodb-org` and can be obtained [here](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu).

**Mongo Express** is an [open source](https://github.com/mongo-express/mongo-express) lightweight web-based administrative interface deployed to manage MongoDB databases interactively. You can access `mongo-express` at `http://localhost:8081`.

!!! Important
    We will be installing **MongoDB** with a _single node/server_ configuration. Note that MongoDB allows for other topologies such as as _replica sets_ that we won't be covering in this documentation.

## References

!!! Note "BCFtools"
    Danecek P, Bonfield JK, et al. Twelve years of SAMtools and BCFtools. Gigascience (2021) 10(2):giab008 [link](https://pubmed.ncbi.nlm.nih.gov/33590861)

!!! Note "SnpEff"
    "A program for annotating and predicting the effects of single nucleotide polymorphisms, SnpEff: SNPs in the genome of Drosophila melanogaster strain w1118; iso-2; iso-3.", Cingolani P, Platts A, Wang le L, Coon M, Nguyen T, Wang L, Land SJ, Lu X, Ruden DM. Fly (Austin). 2012 Apr-Jun;6(2):80-92. PMID: 22728672.

!!! Note "SnpSift"
    "Using Drosophila melanogaster as a model for genotoxic chemical mutational studies with a new program, SnpSift", Cingolani, P., et. al., Frontiers in Genetics, 3, 2012.

!!! Note "dbNSFP v4"
    1. Liu X, Jian X, and Boerwinkle E. 2011. dbNSFP: a lightweight database of human non-synonymous SNPs and their functional predictions. Human Mutation. 32:894-899.
    2. Liu X, Jian X, and Boerwinkle E. 2013. dbNSFP v2.0: A Database of Human Non-synonymous SNVs and Their Functional Predictions and Annotations. Human Mutation. 34:E2393-E2402. 
    3. Liu X, Wu C, Li C, and Boerwinkle E. 2016. dbNSFP v3.0: A One-Stop Database of Functional Predictions and Annotations for Human Non-synonymous and Splice Site SNVs. Human Mutation. 37:235-241. 
    4. Liu X, Li C, Mou C, Dong Y, and Tu Y. 2020. dbNSFP v4: a comprehensive database of transcript-specific functional predictions and annotations for human nonsynonymous and splice-site SNVs. Genome Medicine. 12:103. 
