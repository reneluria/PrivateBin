FROM nginx:1.21

RUN apt-get update && apt-get install -y \
        git \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /var/www/html
WORKDIR /var/www/html

COPY PrivateBin PrivateBin

# RUN git clone https://github.com/PrivateBin/PrivateBin.git && \
#         cd PrivateBin && git checkout 1f872167c181931f866e4902d91b23888b220ad6

#COPY default.conf /etc/nginx/conf.d/default.conf
