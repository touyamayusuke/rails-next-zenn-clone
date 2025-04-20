# syntax = docker/dockerfile:1

ARG RUBY_VERSION=3.1.2
FROM registry.docker.com/library/ruby:$RUBY_VERSION-slim as base

# アプリの作業ディレクトリ
WORKDIR /rails

# 環境変数（開発用）
ENV RAILS_ENV="development" \
    BUNDLE_DEPLOYMENT="0" \
    BUNDLE_PATH="/usr/local/bundle" \
    BUNDLE_WITHOUT=""

# ----------------------------------------
# ビルド用ステージ（gemのインストールなど）
FROM base as build

RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y \
      build-essential \
      default-libmysqlclient-dev \
      libxml2-dev \
      libxslt1-dev \
      git \
      libvips \
      pkg-config

# Gemfile, Gemfile.lock をコピーして bundle install
COPY Gemfile Gemfile.lock ./

RUN gem update --system -N && \
    gem install bundler -v 2.6.8

RUN bundle install && \
    rm -rf ~/.bundle/ "${BUNDLE_PATH}"/ruby/*/cache "${BUNDLE_PATH}"/ruby/*/bundler/gems/*/.git && \
    bundle exec bootsnap precompile --gemfile

# アプリのコードをコピー
COPY . .

# bootsnap の precompile（起動高速化）
RUN bundle exec bootsnap precompile app/ lib/

# ----------------------------------------
# 実行用ステージ（開発用設定）
FROM base

# ⚠️ `build-essential` をここにも含める（←これがないと byebug がビルドできない）
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y \
      curl \
      default-mysql-client \
      libvips \
      build-essential && \
    rm -rf /var/lib/apt/lists /var/cache/apt/archives

# build ステージから成果物をコピー
COPY --from=build /usr/local/bundle /usr/local/bundle
COPY --from=build /rails /rails

# 非rootユーザーの作成と権限付与
RUN useradd rails --create-home --shell /bin/bash && \
    chown -R rails:rails db log storage tmp
USER rails:rails

# ポート開放
EXPOSE 3000

# ログを監視してコンテナを常駐させる（開発向け）
CMD ["bash", "-c", "tail -f log/development.log"]