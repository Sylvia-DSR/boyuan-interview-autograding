# Task 4: AI 扣图工具自动化工作流

# 无需修改以下内容
TOOL_DIR := tool
TOOL_IMAGE_NAME := ai-remover
INPUT_DIR := input_images
OUTPUT_DIR := output_images
ZIP_FILE := processed_images.zip
INPUT_FILES := $(wildcard $(INPUT_DIR)/*.jpg)
OUTPUT_FILES := $(patsubst $(INPUT_DIR)/%.jpg,$(OUTPUT_DIR)/%.png,$(INPUT_FILES))

# 标记文件
DOCKER_BUILD_MARKER := .docker_image_built

.PHONY: all clean process package build shell

all: package

package: $(ZIP_FILE)
	@echo "压缩包已生成在 $(ZIP_FILE)"

$(ZIP_FILE): $(OUTPUT_FILES)
	@echo "创建压缩包 $@"
	@rm -f "$@"
	@if command -v zip >/dev/null 2>&1; then \
		zip -j "$@" $(OUTPUT_FILES); \
	else \
		python3 -c 'import os, sys, zipfile; z = zipfile.ZipFile(sys.argv[1], "w", zipfile.ZIP_DEFLATED); [z.write(p, os.path.basename(p)) for p in sys.argv[2:]]; z.close()' "$@" $(OUTPUT_FILES); \
	fi

# `process' 无需修改
process: $(OUTPUT_FILES)
	@echo "图片都已处理完毕"

clean:
	@rm -rf -- $(OUTPUT_DIR) $(ZIP_FILE) $(DOCKER_BUILD_MARKER)
	@echo "清理完成"

build: $(DOCKER_BUILD_MARKER)

$(DOCKER_BUILD_MARKER): $(TOOL_DIR)/Dockerfile $(TOOL_DIR)/main.py $(TOOL_DIR)/requirements.txt
	@docker build -t $(TOOL_IMAGE_NAME) $(TOOL_DIR)
	@touch $(DOCKER_BUILD_MARKER)

$(OUTPUT_DIR)/%.png: $(INPUT_DIR)/%.jpg $(DOCKER_BUILD_MARKER)
	@echo "正在处理 $< -> $@"
	@mkdir -p $(dir $@)
	@docker run --rm \
		-v "$(CURDIR)/$(INPUT_DIR):/app/input_images" \
		-v "$(CURDIR)/$(OUTPUT_DIR):/app/output_images" \
		-e U2NET_HOME=/app/models \
		$(TOOL_IMAGE_NAME) \
		--input "/app/$<" --output "/app/$@"

# 如果你在使用 Docker 时遇到问题，可以用 `make shell` 进入容器的终端进行调试
shell: $(DOCKER_BUILD_MARKER)
	@docker run --rm -it \
		-v "$(CURDIR)/$(INPUT_DIR):/app/input_images" \
		-v "$(CURDIR)/$(OUTPUT_DIR):/app/output_images" \
		--entrypoint /bin/bash \
		$(TOOL_IMAGE_NAME)