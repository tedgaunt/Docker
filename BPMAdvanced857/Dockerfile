# This docker creates a base installation of BPM Advanced 8.5.7 on CentOS7.
# It is ready to create profiles, using BPMConfig.sh.
#
# To rebuild this image, you will need to obtain and place the 3 installation files from IBM into same directory
# as the Dockerfile:  BPM_Adv_V857_Linux_x86_1_of_3.tar.gz, BPM_Adv_V857_Linux_x86_2_of_3.tar.gz, and BPM_Adv_V857_Linux_x86_3_of_3.tar.gz
#
# because BPM is rather large, make sure that your docker vm is large enough.  create using something like:
# docker-machine create -d virtualbox --virtualbox-disk-size 90000 -virtualbox-memory 5000 --virtualbox-cpu-count 4 default
#
# open a command with the container, and you'll be root.  use "su - wasadmin" before doing anything with WAS/BPM, as that is the owning user.
# future fixpacks and updates should use /home/wasadmin/IBM/InstallationManager/eclipse/tools/imcl

FROM centos:centos7

MAINTAINER Ted Gaunt <Ted.Gaunt@LucidTechnique.com>

ENV BPM_INSTALLER_DIR=/tmp/bpminstaller \
	IBM_DIR=/opt/ibm \
	BPM_DIR=/opt/ibm/bpm \
	WAS_USER=wasadmin \
	WAS_GROUP=wasadmin


# update OS, and add necessary parts
RUN yum update -y \
	&& yum install -q -y unzip ed opensll iputils

# got some of this from IBM docs, but likely don't need some of it when running headless
# RUN yum install -q -y gtk2 libXtst xorg-x11-fonts-Type1

ADD limits.conf bpminstaller/* /tmp/
ADD *.tar.gz /tmp/bpminstaller/
#RUN chmod -R 777 ${BPM_INSTALLER_DIR}

RUN cat /tmp/limits.conf >> /etc/security/limits.conf \
	&& cat /tmp/limits.conf >> /etc/security/limits.d/20-nproc.conf 

#create a user to run everything under.  
RUN groupadd -r ${WAS_GROUP} \
	&& useradd --system -s /bin/bash -m -g ${WAS_GROUP} ${WAS_USER} \
	&& mkdir -p ${IBM_DIR} \
	&& mkdir -p ${BPM_DIR} \
	&& chown -R ${WAS_USER}:${WAS_GROUP} ${IBM_DIR}

USER ${WAS_USER}

# IM is installed as non-admin user, and so data goes to ~wasadmin/IBM
WORKDIR ${BPM_INSTALLER_DIR}/IM64/
RUN ${BPM_INSTALLER_DIR}/IM64/userinst --launcher.ini user-silent-install.ini -log ~/silentinstall-im.log -acceptLicense 

# Now install WAS ND + BPM.  Includes ProcessCenter, which can create profiles for PC or PS
RUN ${BPM_INSTALLER_DIR}/IM64/tools/imcl install \
	com.ibm.bpm.ADV.v85,AdvancedProcessCenter \
	com.ibm.websphere.ND.v85,core.feature,ejbdeploy,thinclient,embeddablecontainer,samples,com.ibm.sdk.6_64bit \
	com.ibm.websphere.IBMJAVA.v70 \
	-acceptLicense \
	-installationDirectory ${BPM_DIR} \
	-repositories ${BPM_INSTALLER_DIR}/repository/repos_64bit/ \
	-showVerboseProgress \
	-log ~/silentinstall-bpm.log

#Switch Java version to 1.7 as per http://www.ibm.com/support/knowledgecenter/SSFPJS_8.5.7/com.ibm.wbpm.imuc.doc/topics/tswitchjavaversion.html?lang=en
RUN ${BPM_DIR}/bin/managesdk.sh -setCommandDefault -sdkname 1.7_64 \
	&& ${BPM_DIR}/bin/managesdk.sh -setNewProfileDefault -sdkname 1.7_64

#cleanup
USER root
RUN rm -rf ${BPM_INSTALLER_DIR} /tmp/*

#TODO:
#add repository to IM  http://public.dhe.ibm.com/software/websphere/bpm/repositories/server/bpmadv/...

#Expose the ports
EXPOSE 2809 9402 9403 9353 9633 9100 11004 11003 9401 7276 7286 5558 5578 5060 5061 8880 9060 9043 9443 9080

# if last user is was_user, then that is what we come in on as.
#USER ${WAS_USER}
#CMD ["/opt/ibm/wlp/bin/server", "run", "defaultServer"]