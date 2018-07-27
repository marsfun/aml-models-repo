FROM index.alauda.cn/alaudaorg/base-modelserver:latest

################################
# extends from parent
# ----------------------------
# EXPOSE gRPC port  8500
# EXPOSE REST port  8501
# MODEL_BASE_PATH=/models
################################

ARG MODEL=chicago-taxi
ADD ${MODEL} ${MODEL_BASE_PATH}/${MODEL}

# override the parent'image env MODEL_NAME
ENV MODEL_NAME=${MODEL}
