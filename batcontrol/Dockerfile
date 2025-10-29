ARG BUILD_FROM
# Refers to the config.yaml
ARG BUILD_VERSION
FROM $BUILD_FROM AS builder

# Install build dependencies
RUN apk add --no-cache \
            python3 \
            py3-pip \
            py3-setuptools \
            git

# Clone the repository at the specified version
RUN git clone --depth 1 --branch ${BUILD_VERSION} https://github.com/muexxl/batcontrol.git /batcontrol

# Set working directory to the cloned repository
WORKDIR /batcontrol

# Build a wheel from the cloned repository
RUN pip wheel --no-cache-dir --no-deps --wheel-dir=/wheels .


# Stage 2: Build the final image
FROM $BUILD_FROM

# Copy the built wheel from the builder stage and install it
COPY --from=builder /wheels /wheels

# Install runtime dependencies
RUN apk add --no-cache \
            python3 \
            py3-pip \
            py3-setuptools \
            git

# Copy the cloned repository from builder stage for config files
COPY --from=builder /batcontrol /batcontrol_source

RUN pip install --no-cache-dir --break-system-packages --extra-index-url https://piwheels.org/simple --prefer-binary /wheels/*.whl


ENV BATCONTROL_VERSION=${BUILD_VERSION}
ENV BATCONTROL_GIT_SHA=${GIT_SHA}
# Set default timezone to UTC, override with -e TZ=Europe/Berlin or similar
# when starting the container
# or set the timezone in docker-compose.yml in the environment section,
ENV TZ=UTC

# Create the app directory and copy the app files
RUN mkdir -p /app /app/logs /app/config
WORKDIR /app

# The load profiles to all locations where it is needed
COPY --from=builder /batcontrol_source/config/load_profile_default.csv ./config/load_profile.csv
COPY --from=builder /batcontrol_source/config/load_profile_default.csv ./default_load_profile.csv

# Copy all the other necessary runtime files
COPY LICENSE /batcontrol_source/entrypoint_ha.sh ./
COPY --from=builder /batcontrol_source/config ./config_template

# Set the scripts as executable
RUN chmod +x entrypoint_ha.sh

# COPY ./src/run_infinite.sh ./run_infinite.sh
# CMD [ "./run_infinite.sh" ]
CMD [ "./entrypoint_ha.sh" ]
