# syntax = docker/dockerfile:1

ARG RUBY_VERSION=3.1.2
FROM registry.docker.com/library/ruby:$RUBY_VERSION-slim as base

# RubyGems をアップデート（nokogiri 対応）
RUN gem update --system 3.4.10

# Rails app lives here
WORKDIR /rails

# Set environment variables
ENV RAILS_ENV="development" \
    BUNDLE_PATH="/usr/local/bundle"

# railsユーザー作成（ここが重要）
RUN useradd rails --create-home --shell /bin/bash && \
    mkdir -p /usr/local/bundle && \
    chown -R rails:rails /usr/local/bundle

# Throw-away build stage to reduce final image size
FROM base as build

# ビルドに必要なパッケージをインストール
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y \
    build-essential \
    default-libmysqlclient-dev \
    git \
    libvips \
    pkg-config \
    libffi-dev \
    libyaml-dev \
    libgmp-dev \
    libssl-dev \
    libreadline-dev \
    zlib1g-dev

# Gemfile をコピーし bundle install
COPY Gemfile Gemfile.lock ./
RUN bundle install && \
    rm -rf ~/.bundle/ "${BUNDLE_PATH}"/ruby/*/cache "${BUNDLE_PATH}"/ruby/*/bundler/gems/*/.git && \
    bundle exec bootsnap precompile --gemfile

# アプリコードをコピーしてプリコンパイル
COPY . .
RUN bundle exec bootsnap precompile app/ lib/

# Final runtime image
FROM base

# 本番用パッケージをインストール
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y \
    curl \
    default-mysql-client \
    libvips && \
    rm -rf /var/lib/apt/lists /var/cache/apt/archives

# ビルド成果物をコピー
COPY --from=build /usr/local/bundle /usr/local/bundle
COPY --from=build /rails /rails

# 必要ディレクトリの所有者を変更
RUN mkdir -p tmp log db storage && \
    chown -R rails:rails /rails /usr/local/bundle

# railsユーザーに切り替え（ここで行う）
USER rails

# EntrypointとCMD
ENTRYPOINT ["/rails/bin/docker-entrypoint"]
EXPOSE 3000
CMD ["./bin/rails", "server"]