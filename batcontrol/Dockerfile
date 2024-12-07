ARG BUILD_FROM
FROM $BUILD_FROM

# Copy data for add-on
RUN apk add --no-cache \
            python3 \
            py3-numpy \
            py3-pandas \
            py3-yaml \
            py3-requests \
            py3-paho-mqtt

COPY ./src /batcontrol
WORKDIR /batcontrol
RUN ln -sf /data/options.json /batcontrol/config/batcontrol_config.yaml

CMD [ "./batcontrol.py" ]
