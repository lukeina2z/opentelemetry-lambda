FROM ubuntu:latest

RUN mkdir /build

COPY . /build

RUN apt-get update && apt-get install -y git curl libssl-dev libreadline-dev zlib1g-dev autoconf bison build-essential libyaml-dev libreadline-dev libncurses5-dev libffi-dev libgdbm-dev zip

RUN  git clone https://github.com/rbenv/rbenv.git ~/.rbenv \
     && git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build \
     && git clone https://github.com/rbenv/rbenv-default-gems.git ~/.rbenv/plugins/rbenv-default-gems \
     && echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.profile \
     && echo 'eval "$(rbenv init -)"' >> ~/.profile \
     && echo 'eval "$(rbenv init -)"' >> ~/.bashrc \
     && echo 'bundler' > ~/.rbenv/default-gems

RUN echo 'alias be="bundle exec"' >> ~/.bashrc
RUN echo 'alias be="bundle exec"' >> ~/.profile

# install rubies to build our gem against Gemfile
RUN . ~/.profile \
   && cd /root/.rbenv/plugins/ruby-build && git pull && cd - \
   && rbenv install 3.2.0 \
   && rbenv install 3.3.0 \
   && rbenv install 3.4.0

WORKDIR /build/layer

RUN . ~/.profile && rbenv local 3.2.0 && bundle install
RUN . ~/.profile && rbenv local 3.3.0 && bundle install
RUN . ~/.profile && rbenv local 3.4.0 && bundle install

WORKDIR /root/.rbenv/versions/3.2.0/lib/ruby/gems/
RUN zip -r gems-3.2.0.zip 3.2.0/

WORKDIR /root/.rbenv/versions/3.3.0/lib/ruby/gems/
RUN zip -r gems-3.3.0.zip 3.3.0/

WORKDIR /root/.rbenv/versions/3.4.0/lib/ruby/gems/

# rbenv install 3.4.0 get 3.4.0+1/, so need to change back to 3.4.0+1
RUN mv 3.4.0+1/ 3.4.0/
RUN set -e && \
  dir=$(find /root/.rbenv/versions/3.4.0/lib/ruby/gems/ -type d -name '3.4.0+1' | head -n 1) && \
  target=$(echo "$dir" | sed 's/3\.4\.0+1/3.4.0/') && \
  mv "$dir" "$target"
RUN zip -r gems-3.4.0.zip 3.4.0/

# copy gems to /build/ruby/gems for zipping 
RUN mkdir /build/ruby && mkdir /build/ruby/gems
WORKDIR /build/ruby/gems
RUN cp /root/.rbenv/versions/3.2.0/lib/ruby/gems/gems-3.2.0.zip . && unzip gems-3.2.0.zip && rm gems-3.2.0.zip
RUN cp /root/.rbenv/versions/3.3.0/lib/ruby/gems/gems-3.3.0.zip . && unzip gems-3.3.0.zip && rm gems-3.3.0.zip
RUN cp /root/.rbenv/versions/3.4.0/lib/ruby/gems/gems-3.4.0.zip . && unzip gems-3.4.0.zip && rm gems-3.4.0.zip
RUN ls -al /build/ruby/gems

# rm gem cache
RUN rm /root/.rbenv/versions/3.2.0/lib/ruby/gems/3.2.0/cache/* \
    && rm /root/.rbenv/versions/3.3.0/lib/ruby/gems/3.3.0/cache/* \
    && rm /root/.rbenv/versions/3.4.0/lib/ruby/gems/3.4.0/cache/*

# zip all the gems
WORKDIR /build
RUN cp layer/otel-handler . && cp layer/wrapper.rb .
RUN zip -r opentelemetry-ruby-layer.zip ruby/ otel-handler wrapper.rb

CMD cp /build/opentelemetry-ruby-layer.zip /out/opentelemetry-ruby-layer.zip
