FROM carello/virlutils:v1
COPY . /app
WORKDIR /app
RUN chmod +x /app/virlstart.sh

#CMD [ "/bin/sh", "-c", "./virlstart.sh" ]
#ENTRYPOINT [ "/bin/sh", "virl up -f topology.virl" ]
#CMD [ "/bin/sh", "-c", "virl up -f topology.virl" ]

ENTRYPOINT /app/virlstart.sh
