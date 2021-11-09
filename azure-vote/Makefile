# Optional - Run the setup in local only
setup:
	# Create python virtualenv & source it
	# source ~/.devops/bin/activate
	python3 -m venv ~/.devops
	
install:
	# Install dependencies	
	pip install --upgrade pip &&\
		pip install -r requirements.txt
	# Install hadolint
	wget -O /bin/hadolint https://github.com/hadolint/hadolint/releases/download/v2.8.0/hadolint-Linux-x86_64 &&\
		chmod +x /bin/hadolint

lint:	
	hadolint Dockerfile
	pylint --disable=R,C,W1309,W1203,W1202,W1201 ./azure-vote/main.py

all: install lint test