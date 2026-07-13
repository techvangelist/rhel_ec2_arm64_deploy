.PHONY: help install build-ee push-ee deploy configure verify all clean lint

# Variables
EE_NAME ?= rhel10-ec2-ee
EE_VERSION ?= latest
REGISTRY ?= quay.io
REGISTRY_ORG ?= your-org
EE_IMAGE = $(REGISTRY)/$(REGISTRY_ORG)/$(EE_NAME):$(EE_VERSION)

help:
	@echo "Available targets:"
	@echo "  install      - Install Ansible collections and Python dependencies"
	@echo "  build-ee     - Build the Execution Environment container image"
	@echo "  push-ee      - Push the Execution Environment to registry"
	@echo "  deploy       - Deploy EC2 instance"
	@echo "  configure    - Configure EC2 instance (requires INSTANCE_IP)"
	@echo "  verify       - Verify EC2 instance configuration (requires INSTANCE_IP)"
	@echo "  all          - Deploy, configure, and verify (complete workflow)"
	@echo "  clean        - Clean temporary files and caches"
	@echo "  lint         - Run ansible-lint on playbooks and roles"
	@echo ""
	@echo "Examples:"
	@echo "  make install"
	@echo "  make build-ee"
	@echo "  make deploy"
	@echo "  make configure INSTANCE_IP=18.223.210.38"
	@echo "  make all"

install:
	@echo "Installing Ansible collections..."
	ansible-galaxy collection install -r requirements.yml --force
	@echo "Installing Python dependencies..."
	pip install -r requirements.txt
	@echo "Installation complete!"

build-ee:
	@echo "Building Execution Environment: $(EE_IMAGE)"
	ansible-builder build -t $(EE_NAME):$(EE_VERSION) -v 3
	@echo "Tagging for registry..."
	podman tag $(EE_NAME):$(EE_VERSION) $(EE_IMAGE)
	@echo "Build complete: $(EE_IMAGE)"

push-ee:
	@echo "Pushing $(EE_IMAGE) to registry..."
	@podman login $(REGISTRY)
	podman push $(EE_IMAGE)
	@echo "Push complete!"

deploy:
	@echo "Deploying RHEL 10 ARM64 EC2 instance..."
	ansible-playbook deploy.yml
	@echo "Deployment complete!"

configure:
ifndef INSTANCE_IP
	@echo "ERROR: INSTANCE_IP is required"
	@echo "Usage: make configure INSTANCE_IP=<ip-address>"
	@exit 1
endif
	@echo "Configuring instance at $(INSTANCE_IP)..."
	ansible-playbook -i $(INSTANCE_IP), configure.yml
	@echo "Configuration complete!"

verify:
ifndef INSTANCE_IP
	@echo "ERROR: INSTANCE_IP is required"
	@echo "Usage: make verify INSTANCE_IP=<ip-address>"
	@exit 1
endif
	@echo "Verifying instance at $(INSTANCE_IP)..."
	ansible-playbook -i $(INSTANCE_IP), verify.yml
	@echo "Verification complete!"

all:
	@echo "Running complete workflow (deploy, configure, verify)..."
	ansible-playbook site.yml
	@echo "Complete workflow finished!"

clean:
	@echo "Cleaning temporary files..."
	find . -type f -name '*.retry' -delete
	find . -type d -name '__pycache__' -exec rm -rf {} + 2>/dev/null || true
	find . -type f -name '*.pyc' -delete
	rm -rf /tmp/ansible_facts/
	rm -rf context/
	rm -rf collections/
	@echo "Clean complete!"

lint:
	@echo "Running ansible-lint..."
	ansible-lint deploy.yml configure.yml verify.yml site.yml roles/
	@echo "Lint complete!"
