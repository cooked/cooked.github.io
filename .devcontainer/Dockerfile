FROM ruby:2.7-alpine

RUN gem install bundler -v 2.4.22

RUN apk --no-cache add build-base git
#make gcc g++ libc-dev libffi-dev

# # https://talk.jekyllrb.com/t/load-error-cannot-load-such-file-webrick/5417/2
#RUN gem update bundler && gem install bundler jekyll

COPY Gemfile ./
RUN bundle install

## Set up user so that we can share ssh credentials.
# see: https://stackoverflow.com/questions/27701930/how-to-add-users-to-docker-container 
ARG DOCKUSER=jekyll
RUN addgroup -S $DOCKUSER --gid 1000 && \
    adduser -S $DOCKUSER -s /bin/sh --disabled-password --uid 1000 -G $DOCKUSER
USER $DOCKUSER

# web and livereload ports
EXPOSE 4000
EXPOSE 35729

CMD /bin/bash