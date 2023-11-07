ARG BUILD_FROM
FROM $BUILD_FROM

# Copy data for add-on
RUN apk add --no-cache \
            python3 \
            py3-numpy \
            py3-pandas\
            py3-yaml\
            py3-requests

COPY ./batcontrol /batcontrol
WORKDIR /batcontrol
RUN ln -s /data/options.json /batcontrol/config/batcontrol_config.yaml

CMD [ "./batcontrol.py" ]
