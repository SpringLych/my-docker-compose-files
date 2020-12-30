ARG ELK_VERSION

# https://www.docker.elastic.co/
# FROM docker.elastic.co/elasticsearch/elasticsearch:${ELK_VERSION}
FROM docker.elastic.co/elasticsearch/elasticsearch:7.9.3

# Add your elasticsearch plugins setup here
# Example: RUN elasticsearch-plugin install analysis-icu
# 安装中文分词插件
RUN elasticsearch-plugin install -b https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.9.3/elasticsearch-analysis-ik-7.9.3.zip