FROM python:3.6-slim

RUN apt update \
    && apt install -y --no-install-recommends wget \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /tmp
RUN wget --no-check-certificate https://github.com/gohugoio/hugo/releases/download/v0.52/hugo_0.52_Linux-64bit.deb \
    && dpkg -i hugo_0.52_Linux-64bit.deb

RUN pip install rst2html5 \
    && export PATH="/bin:${PATH}"


# Expose default hugo port
EXPOSE 1313

# You must bind the blog directory to /app
WORKDIR /app
ENV HUGO_BASE_URL http://localhost:1313
CMD hugo server -b ${HUGO_BASE_URL} --bind=0.0.0.0 --verbose

# How to use:
# For some reason, I couldn't bind other ports than 1313 from host.
# So just bind 1313.

# docker run -v ~/blog/:/app -i -t --rm -p 1313:1313 shinyeyes/hugo:v0.52