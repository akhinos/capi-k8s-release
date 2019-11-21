FROM ruby:2.5.5 as builder

RUN apt-get update && apt-get install -y git \
  libxml2-dev \
  libxslt-dev \
  libmariadb-dev \
  tzdata \
  libpq-dev \
  jq \
  less \
  dnsutils

COPY cloud_controller_ng/Gemfile .
COPY cloud_controller_ng/Gemfile.lock .
COPY cloud_controller_ng/ .

RUN mkdir lifecycle_hooks
COPY lifecycle_hooks/ lifecycle_hooks/

RUN bundle config build.nokogiri --use-system-libraries
RUN bundle install

FROM cloudfoundry/run:tiny

ENV BUNDLE_GEMFILE /cloud_controller_ng/Gemfile
ENV CLOUD_CONTROLLER_NG_CONFIG /config/cloud_controller_ng.yml
ENV C_INCLUDE_PATH /libpq/include
ENV DYNO #{spec.job.name}-#{spec.index}
ENV LANG en_US.UTF-8
ENV LIBRARY_PATH /libpq/lib
ENV RAILS_ENV production

COPY --from=builder /lib/x86_64-linux-gnu/libz.so.* /lib/x86_64-linux-gnu/
COPY --from=builder /usr/lib/x86_64-linux-gnu/libyaml* /usr/lib/x86_64-linux-gnu/
COPY --from=builder /usr/lib/x86_64-linux-gnu/libgmp* /usr/lib/x86_64-linux-gnu/
COPY --from=builder /usr/local/lib/ /usr/local/lib
COPY --from=builder /usr/local/bin/ /usr/local/bin

COPY --from=builder cloud_controller_ng/ .
COPY --from=builder lifecycle_hooks/ .

ENTRYPOINT ["/cloud_controller_ng/bin/cloud_controller", "-c", "/config/cloud_controller_ng.yml"]