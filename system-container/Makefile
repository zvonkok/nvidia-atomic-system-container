USERNAME := zvonkok
REPO := redhat
VERSION := $(shell rpm -qv nvidia-kmod | cut -d '-' -f3)
KERNEL := $(shell uname -r)

hook: exports/hostfs/usr/libexec/oci/hooks.d/oci-nvidia-container-runtime

exports/hostfs/usr/libexec/oci/hooks.d/oci-nvidia-container-runtime: oci-nvidia-container-runtime
	mkdir -p $(dir $@)	
	cp $< $@

exports/config.json.template: Makefile
	printf '{"process" : {"args" : []}, "root" : {"path" : "rootfs", "readonly" : true}\n\n}' > $@

exports/manifest.json: manifest.json
	cp  $< $@

exports/hostfs/etc/dracut.conf.d/nvidia.conf: Makefile
	mkdir -p $(dir $@)
	printf "echo drivers_dir+=/opt/nvidia/lib/modules/$(uname -r)/\n" > $@

exports/hostfs/opt/nvidia: Makefile
	mkdir -p $@/bin
	mkdir -p $@/lib
	mkdir -p $@/etc/nvidia-container-runtime

exports/hostfs/etc/ld.so.conf.d/nvidia.conf: Makefile
	mkdir -p $(dir $@)	
	@echo "/opt/nvidia/lib" > $@

exports/hostfs/etc/profile.d/nvidia.sh: nvidia.sh
	mkdir -p $(dir $@)
	cp $< $@

exports/hostfs/opt/nvidia/bin/nvidia-mkdev: Makefile
	mkdir -p $(dir $@)
	cp -a nvidia-mkdev $@

exports/hostfs/etc/modprobe.d/nouveau.conf: Makefile
	mkdir -p $(dir $@)
	@echo "blacklist nouveau" > $@
	@echo "install nouveau /bin/true" >> $@


modules.order: Makefile
	touch exports/hostfs/opt/nvidia/lib/modules/$(KERNEL)/$@
modules.builtin: Makefile
	touch exports/hostfs/opt/nvidia/lib/modules/$(KERNEL)/$@
modules.dep: Makefile
	depmod -b exports/hostfs/opt/nvidia

exports: exports/hostfs/opt/nvidia \
	exports/manifest.json \
	exports/config.json.template \
	exports/hostfs/etc/dracut.conf.d/nvidia.conf \
	exports/hostfs/etc/ld.so.conf.d/nvidia.conf  \
	exports/hostfs/etc/profile.d/nvidia.sh  \
	exports/hostfs/etc/modprobe.d/nouveau.conf \
	exports/hostfs/opt/nvidia/bin/nvidia-mkdev 

all: exports hook modules.order modules.builtin modules.dep
	docker build -t $(USERNAME)/$(REPO)\:latest .
	docker tag $(USERNAME)/$(REPO)\:latest docker.io/$(USERNAME)/$(REPO)\:nvidia-$(VERSION)-kernel-$(KERNEL)

push: all
	docker login --username zvonkok --password='' docker.io
	docker push docker.io/zvonkok/redhat:nvidia-$(VERSION)-kernel-$(KERNEL)

clean: 
	rm -rf exports

.PHONY: clean all exports hook

# yum install atomic 
# docker login --username zvonkok --password='o2lZcl!On1oCCN&g' docker.io
# atomic pull --storage ostree docker.io/zvonkok/redhat:nvidia-384.81
# atomic install --system --system-package=no zvonkok/redhat:nvidia-384.81 
# ldconfig




