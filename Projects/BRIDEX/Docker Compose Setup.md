```
version: "3.9"
services:
  fe:
    container_name: bx-fe
    image: vitrio/bridex:fe
    ports:
      - 41313:11113
    volumes:
      - ./temp:/temp
      - ./fe:/bridex-fe
    networks:
      bridex:

  ndp:
    container_name: bx-ndp
    image: vitrio/bridex:ndp
    ports:
      - 41444:11115
    volumes:
      - ./temp:/temp
      - ./be-ndp:/bridex-ndp
    links:
      - db
    networks:
      bridex:

  dp:
    container_name: bx-dp
    image: vitrio/bridex:dp
    ports:
      - 41333:11117
    volumes:
      - ./temp:/temp
      - ./be-dp:/bridex-dp
    links:
      - db
    networks:
      bridex:

  py:
    container_name: bx-py
    image: jupyter/datascience-notebook
    ports:
      - 10000:11119
    volumes:
      - ./temp:/temp
      - ./py:/home/jovyan/.jupyter
    networks:
      bridex:

  db:
    container_name: bx-db
    image: singlestore/cluster-in-a-box
    ports:
      - 41133:3306
      - 41141:8080
    volumes:
      - db-data:/var/lib/memsql
      #- ./data:/data:ro
      - ./init.sql:/init.sql
    environment:
      - LICENSE_KEY=BDRjODdlNWVmYzk2NTRlNDhiODJjZGEwYzQ5MjA5ZDQ1AAAAAAAAAAAEAAAAAAAAAAwwNQIZAIFk+F4FUDmLggb9TpctFxRUOSFD4UPmxwIYO22uIWGwW4vmISxDW4LziZgyPVg/6psHAA==
      - START_AFTER_INIT=Y
      - ROOT_PASSWORD=akito123
    networks:
      bridex:

volumes:
  db-data:

networks:
  bridex:
    driver: bridge
    ipam:
      config:
        - subnet: 172.19.0.0/24
          gateway: 172.19.0.1

```