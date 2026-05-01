# Options are stable | alpha | beta
CHANNEL := stable

# Make sure to https://www.flatcar.org/docs/latest/installing/vms/qemu/#gentoo

BUTANE = $(shell find butane -type f -not -name 'node*' | sort -h) $(shell find butane -type f -name "node*.yaml")
IGN = $(patsubst butane/%.yaml,ign/%.ign,$(BUTANE))

ifeq ($(shell command -v podman 2> /dev/null),)
    CR=docker
else
    CR=podman
endif

# Remove files that we can easily recreate
clean:
	rm -f flatcar_production_qemu_image.img.sig config.json ign/* /var/lib/libvirt/images/flatcar/*.ign

# Remove files that take longer to recreate/download
cleanclean: clean
	rm -f flatcar_production_qemu_image.img.fresh Flatcar_Image_Signing_Key.asc $(TOKEN_FILE)

.PHONY: cleanclean clean

Flatcar_Image_Signing_Key.asc:
	curl -L -O https://www.flatcar.org/security/image-signing-key/Flatcar_Image_Signing_Key.asc

verify-gpg: Flatcar_Image_Signing_Key.asc
	gpg --import --keyid-format LONG Flatcar_Image_Signing_Key.asc

.PHONY: verify-gpg

# Download and verify the image that we'll use as a base
flatcar_production_qemu_image.img.fresh:
	wget "https://${CHANNEL}.release.flatcar-linux.net/amd64-usr/current/flatcar_production_qemu_image.img"
	wget "https://${CHANNEL}.release.flatcar-linux.net/amd64-usr/current/flatcar_production_qemu_image.img.sig"
	gpg --verify flatcar_production_qemu_image.img.sig
	cp -f flatcar_production_qemu_image.img flatcar_production_qemu_image.img.fresh

# Create vm disks from the base image
%.qcow2: flatcar_production_qemu_image.img.fresh
	mkdir -p /var/lib/libvirt/images/flatcar
	qemu-img create -f qcow2 -F qcow2 -b $(shell pwd)/flatcar_production_qemu_image.img.fresh /var/lib/libvirt/images/flatcar/$@

# Convert butane configs to ignition configs
ign/%.ign:
	mkdir -p ign
	$(CR) run --rm -i -v $(shell pwd)/butane:/config/butane:ro -v $(shell pwd)/ign:/config/ign quay.io/coreos/butane:latest --pretty -d /config < butane/$*.yaml > $@

# Create VMs
virt-node%: flatcar_production_qemu_image-%.qcow2 $(IGN)
	cp -fv ign/*.ign /var/lib/libvirt/images/flatcar
	virt-install --connect qemu:///system --import --name $@  --ram 4096  --vcpus 4 --os-variant=generic --network network=default,model=virtio --disk path=/var/lib/libvirt/images/flatcar/flatcar_production_qemu_image-$*.qcow2,format=qcow2,bus=virtio --vnc --qemu-commandline="-fw_cfg name=opt/org.flatcar-linux/config,file=/var/lib/libvirt/images/flatcar/node$*.ign" --noautoconsole

# Don't delete intermediate files
.PRECIOUS: %.qcow2 ign/%.ign

# Stop and Remove VMs
rm-virt-node%:
	virsh destroy virt-node$* || :
	virsh undefine virt-node$*

# Create all ignition configs
new-cluster: clean $(IGN)

.PHONY: ha rm-ha
