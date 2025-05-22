ARG BUILD_FROM
FROM $BUILD_FROM AS builder
# Copy only whats needed for dependencies first
COPY ./src/pyproject.toml ./src/LICENSE ./src/README.MD ./

# Copy data for add-on
RUN apk add --no-cache \
            python3 \
            py3-pip \
            py3-setuptools



# RUN pip install --no-cache-dir setuptools>=66.0


# Copy the rest of the source files
COPY ./src/src ./src


# Build a wheel from the public Git repo
RUN pip wheel --no-cache-dir --no-deps --wheel-dir=/wheels .


# Install build dependencies

# Stage 2: Build the final image
FROM $BUILD_FROM

# Copy the built wheel from the builder stage and install it
COPY --from=builder /wheels /wheels

RUN apk add --no-cache \
            python3 \
            py3-pip \
            py3-setuptools
RUN pip install --no-cache-dir --break-system-packages --extra-index-url https://piwheels.org/simple --prefer-binary /wheels/*.whl
# RUN rm -rf /wheels


ENV BATCONTROL_VERSION=${VERSION}
ENV BATCONTROL_GIT_SHA=${GIT_SHA}
# Set default timezone to UTC, override with -e TZ=Europe/Berlin or similar
# when starting the container
# or set the timezone in docker-compose.yml in the environment section,
ENV TZ=UTC

# Create the app directory and copy the app files
RUN mkdir -p /app /app/logs /app/config
WORKDIR /app

# The load profiles to all locations where it is needed
COPY src/config/load_profile_default.csv ./config/load_profile.csv
COPY src/config/load_profile_default.csv ./default_load_profile.csv

# Copy all the other necessary runtime files
COPY LICENSE src/entrypoint_ha.sh ./
COPY src/config ./config_template

# Set the scripts as executable
RUN chmod +x entrypoint_ha.sh

# COPY ./src/run_infinite.sh ./run_infinite.sh
# CMD [ "./run_infinite.sh" ]
CMD [ "./entrypoint_ha.sh" ]
