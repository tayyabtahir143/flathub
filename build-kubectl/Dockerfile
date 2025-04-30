FROM docker:24.0.7-cli

# Install curl, bash, kubectl
RUN apk add --no-cache curl bash \
    && curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl \
    && install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl \
    && rm kubectl

CMD ["kubectl"]

