FROM registry.access.redhat.com/rhscl/s2i-base-rhel7

# This image provides a Node.JS environment you can use to run your Node.JS
# applications.

EXPOSE 8080 5000

# Add $HOME/node_modules/.bin to the $PATH, allowing user to make npm scripts
# available on the CLI without using npm's --global installation mode
# This image will be initialized with "npm run $NPM_RUN"
# See https://docs.npmjs.com/misc/scripts, and your repo's package.json
# file for possible values of NPM_RUN
ENV NODEJS_VERSION=6.9.1 \
    NPM_RUN=start \
    NPM_CONFIG_PREFIX=$HOME/.npm-global \
    PATH=$HOME/node_modules/.bin/:$HOME/.npm-global/bin/:$PATH

LABEL summary="Platform for building and running Node.js 6 applications" \
      io.k8s.description="Platform for building and running Node.js 6 applications" \
      io.k8s.display-name="Node.js 6" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,nodejs,nodejs6" \
      com.redhat.dev-mode="DEV_MODE:false" \
      com.redhat.deployments-dir="/opt/app-root/src" \
      com.redhat.dev-mode.port="DEBUG_PORT:5858"

# Labels consumed by Red Hat build service
LABEL com.redhat.component="nodejs6" \
      name="rhscl/nodejs-6-rhel7" \
      version="6.9" \
      release="1" \
      architecture="x86_64"

# To use subscription inside container yum command has to be run first (before yum-config-manager)
# https://access.redhat.com/solutions/1443553
RUN yum repolist > /dev/null && \
    yum-config-manager --enable rhel-server-rhscl-7-rpms && \
    yum-config-manager --enable rhel-7-server-optional-rpms && \
    yum-config-manager --enable rhel-7-server-extras-rpms && \
    yum-config-manager --enable rhel-7-server-ose-3.3-rpms && \
    yum-config-manager --disable epel >/dev/null || : && \

    #These packages will come from the Satellite server
    INSTALL_PKGS="bzip2 nss_wrapper gcc-c++ make" && \

    #Get Node RPM
    #Create the Node RPM package name
    NODEJS_RPM="nodejs-${NODEJS_VERSION}-1nodesource.el7.centos.x86_64.rpm" && \
    #Download the node RPM package name
    curl -ks -O https://rpm.nodesource.com/pub_6.x/el/7/x86_64/${NODEJS_RPM} && \
    #Verify the rpm package prior to install
    rpm -qpi ${NODEJS_RPM} && \
    rpm -Kvv --nosignature ${NODEJS_RPM}

    #Install the packages.
    yum install -y --setopt=tsflags=nodocs $INSTALL_PKGS $NODEJS_RPM && \
    #Verify the packages
    rpm -V $INSTALL_PKGS && \
    #Always do this
    yum clean all -y

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

# Each language image can have 'contrib' a directory with extra files needed to
# run and build the applications.
COPY ./contrib/ /opt/app-root

# Drop the root user and make the content of /opt/app-root owned by user 1001
RUN chown -R 1001:0 /opt/app-root && chmod -R ug+rwx /opt/app-root && \
    chown -R 1001:0 $STI_SCRIPTS_PATH && chmod -R ug+rwx $STI_SCRIPTS_PATH

USER 1001

# Set the default CMD to print the usage of the language image
CMD $STI_SCRIPTS_PATH/usage
