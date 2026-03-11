---
title: ClawBot Backup Skill 学习笔记
author: ClawBot 社区
source: https://github.com/clawdbot/backup-skill
date: 2026-03-11
tags: [OpenClaw, 备份恢复, Git版本控制, 自动化]
category: 系统维护
---

# ClawBot Backup Skill 学习笔记

## 📋 技能概述

**功能**：备份、恢复、同步 OpenClaw 配置和技能
**目标**：跨设备同步、版本控制、自动化备份、迁移到新机器
**作者**：ClawBot 社区
**评分**：⭐⭐⭐⭐⭐（非常全面）

---

## 🎯 核心功能

### 1. 备份

**全量备份**：
- 备份所有数据：skills/、commands/、settings.json、mcp/、contexts/、templates/
- 打包成 timestamp 的 tar.gz 文件
- 支持筛选备份类型（full/skills/settings）

**增量备份**：
- 只备份修改过的文件（基于文件修改时间）
- 节省存储空间和备份时间

### 2. 恢复

**全量恢复**：
- 从备份文件恢复所有数据
- 支持预览备份内容（不真正恢复）
- 验证备份完整性

**选择性恢复**：
- 只恢复某个目录（skills/、commands/）
- 只恢复某个文件（settings.json）

### 3. 同步

**Git 同步**：
- 用 Git 管理配置和技能
- 自动提交变更
- 推送到远程仓库
- 支持多设备克隆和同步

**Rsync 同步**：
- 同步到远程服务器
- 支持增量同步（只传输修改过的文件）
- 自动清理远程多余的文件

**云存储同步**：
- 同步到 Dropbox/Google Drive
- 简单的文件复制命令

### 4. 自动化

**Cron 定时任务**：
- 每天凌晨 2:00 全量备份
- 每周日清理旧备份
- 每 6 小时自动 commit 到 Git

**Systemd 定时器**：
- 更现代的 Linux 定时系统
- 支持更复杂的时间表（OnCalendar=daily）
- 支持持久化（Persistent=true）

**Launchd（macOS）**：
- macOS 原生的定时系统
- plist 文件配置
- 适合 macOS 用户

---

## 🔄 备份策略对比

### OpenClaw vs ClawBot

| 维度 | OpenClaw | ClawBot |
|------|---------|----------|
| **核心目录** | memory/, skills/, workspace/, logs/ | skills/, commands/, settings.json, mcp/, contexts/, templates/ |
| **备份方式** | tar.gz 打包 | tar.gz / Git / Rsync |
| **版本控制** | 无 | Git |
| **跨设备同步** | 无 | Git / Rsync / 云存储 |
| **自动化** | Cron | Cron / Systemd / Launchd |
| **增量备份** | 基于修改时间（24小时） | 基于修改时间 + Git 增量 |
| **清理策略** | 7天/30天 | 最近 N 个备份（可配置） |

---

## 💡 可借鉴的理念

### 1. Git 版本控制

**为什么好**：
- 完整的历史记录（每次修改都可追溯）
- 方便的分支管理（实验性修改不影响主分支）
- 多设备协作（如果团队共享技能）
- 冲突解决机制（如果多个设备同时修改）

**如何应用到 OpenClaw**：
```bash
cd ~/.openclaw

# 初始化 Git
git init

# 创建 .gitignore
cat > .gitignore << 'EOF'
# 机器特定设置
openclaw.json.local

# 缓存和临时文件
__pycache__/
*.pyc
*.tmp
*.log

# 大文件
workspace/temp/

# 敏感数据
*.pem
*.key
credentials/
EOF

# 初始提交
git add .
git commit -m "Initial OpenClaw configuration"
```

### 2. 智能清理策略

**ClawBot 的策略**：
- 保留最近 N 个备份（默认 10 个）
- 超过 N 个的自动删除
- 防止备份目录无限增长

**应用到 OpenClaw**：
```bash
# 保留最近 7 个全量备份
find ~/backups/openclaw -maxdepth 1 -type d -name "full_*" \
    | sort -r | tail -n +8 | xargs rm -rf

# 保留最近 30 个增量备份
find ~/backups/openclaw -maxdepth 1 -type d -name "incremental_*" \
    | sort -r | tail -n +31 | xargs rm -rf
```

### 3. 备份验证机制

**ClawBot 的验证**：
- 恢复前预览备份内容
- 验证备份完整性（tar -tzvf）
- 对比备份与当前状态

**应用到 OpenClaw**：
```bash
# 验证备份完整性
tar -tzvf ~/backups/openclaw/full_20260311_145750/memory.tar.gz \
    > /dev/null && echo "备份 OK" || echo "备份损坏"

# 列出备份内容
tar -tzvf ~/backups/openclaw/full_20260311_145750/memory.tar.gz \
    | head -20

# 对比备份与当前状态
diff -rq ~/.openclaw/memory \
    ~/backups/openclaw/restore_test/memory
```

---

## 🚀 增强版 OpenClaw 备份脚本

基于 ClawBot 的理念，我创建了一个增强版的备份脚本：

```bash
#!/bin/bash
# openclaw-backup.sh - 增强版 OpenClaw 备份脚本
# 借鉴 ClawBot Backup Skill 的理念

set -e

# 配置
BACKUP_DIR="${HOME}/backups/openclaw"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
LOG_FILE="${BACKUP_DIR}/backup_${TIMESTAMP}.log"
OPENCLAW_DIR="${HOME}/.openclaw"
CLONE_DIR="${HOME}/.openclaw-clone" # 用于恢复演练

# 备份保留策略
MAX_FULL_BACKUPS=7        # 保留最近 7 个全量备份
MAX_INCREMENTAL_BACKUPS=30 # 保留最近 30 个增量备份
MAX_LOG_FILES=90          # 保留最近 90 个日志文件

# 颜色输出
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m'

# 日志函数
log() {
    echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')]${NC} $1" | tee -a "${LOG_FILE}"
}

error() {
    echo -e "${RED}[ERROR]${NC} $1" | tee -a "${LOG_FILE}"
    exit 1
}

warning() {
    echo -e "${YELLOW}[WARNING]${NC} $1" | tee -a "${LOG_FILE}"
}

success() {
    echo -e "${GREEN}[SUCCESS]${NC} $1" | tee -a "${LOG_FILE}"
}

# 检查依赖
check_dependencies() {
    log "检查依赖..."
    
    if ! command -v tar &> /dev/null; then
        error "tar 命令未找到，请先安装 tar"
    fi
    
    if ! command -v git &> /dev/null; then
        warning "git 命令未找到，版本控制功能将不可用"
    fi
    
    success "依赖检查完成"
}

# 创建备份目录
create_backup_dir() {
    mkdir -p "${BACKUP_DIR}"
    mkdir -p "${CLONE_DIR}"
}

# 全量备份
full_backup() {
    log "开始全量备份..."
    
    BACKUP_PATH="${BACKUP_DIR}/full_${TIMESTAMP}"
    mkdir -p "${BACKUP_PATH}"
    
    # 备份核心目录
    log "备份核心目录..."
    for dir in memory skills workspace logs; do
        if [ -d "${OPENCLAW_DIR}/${dir}" ]; then
            log "  备份 ${dir}/"
            tar -czf "${BACKUP_PATH}/${dir}.tar.gz" -C "${OPENCLAW_DIR}" "${dir}"
            success "  ${dir}/ 已备份"
        else
            warning "  ${dir}/ 不存在，跳过"
        fi
    done
    
    # 备份核心文件
    log "备份核心文件..."
    for file in HEARTBEAT_INSTREET.md daily-todo.md; do
        if [ -f "${OPENCLAW_DIR}/${file}" ]; then
            log "  备份 ${file}"
            cp "${OPENCLAW_DIR}/${file}" "${BACKUP_PATH}/${file}"
            success "  ${file} 已备份"
        else
            warning "  ${file} 不存在，跳过"
        fi
    done
    
    # 生成备份清单
    generate_backup_manifest "${BACKUP_PATH}"
    
    success "全量备份完成：${BACKUP_PATH}"
    log "备份文件："
    ls -lh "${BACKUP_PATH}" | grep -v "total" | tee -a "${LOG_FILE}"
}

# 增量备份
incremental_backup() {
    log "开始增量备份..."
    
    BACKUP_PATH="${BACKUP_DIR}/incremental_${TIMESTAMP}"
    mkdir -p "${BACKUP_PATH}"
    
    # 检查上次备份时间
    LAST_BACKUP_FILE="${BACKUP_DIR}/.last_backup_time"
    LAST_BACKUP_TIME=0
    if [ -f "${LAST_BACKUP_FILE}" ]; then
        LAST_BACKUP_TIME=$(cat "${LAST_BACKUP_FILE}")
    fi
    
    # 当前时间戳
    CURRENT_TIME=$(date +%s)
    
    # 备份修改过的文件（过去 24 小时内修改过的）
    log "检查修改过的文件（过去 24 小时）..."
    MODIFIED_FILES=0
    
    # 检查核心文件
    for file in HEARTBEAT_INSTREET.md daily-todo.md; do
        if [ -f "${OPENCLAW_DIR}/${file}" ]; then
            FILE_MTIME=$(stat -f %m "${OPENCLAW_DIR}/${file}")
            FILE_TIME=$(date -r "${OPENCLAW_DIR}/${file}" +%s)
            
            TIME_DIFF=$((CURRENT_TIME - FILE_TIME))
            if [ ${TIME_DIFF} -lt 86400 ]; then
                log "  备份修改过的文件：${file}"
                cp "${OPENCLAW_DIR}/${file}" "${BACKUP_PATH}/${file}.${TIMESTAMP}"
                MODIFIED_FILES=$((MODIFIED_FILES + 1))
                success "  ${file}.${TIMESTAMP} 已备份"
            fi
        fi
    done
    
    # 记录本次备份时间
    echo "${TIMESTAMP}" > "${LAST_BACKUP_FILE}"
    
    if [ ${MODIFIED_FILES} -eq 0 ]; then
        warning "没有文件在过去 24 小时内修改过，增量备份完成（0 个文件）"
    else
        success "增量备份完成（${MODIFIED_FILES} 个文件）"
    fi
    
    log "备份文件："
    ls -lh "${BACKUP_PATH}" | grep -v "total" | tee -a "${LOG_FILE}"
}

# Git 版本控制（可选）
git_backup() {
    log "开始 Git 备份..."
    
    cd "${OPENCLAW_DIR}" || return
    
    # 检查是否是 Git 仓库
    if [ ! -d ".git" ]; then
        log "初始化 Git 仓库..."
        git init
        
        # 创建 .gitignore
        cat > .gitignore << 'EOF'
# 机器特定设置
*.local

# 缓存和临时文件
__pycache__/
*.pyc
*.tmp
*.log

# 大文件
workspace/temp/

# 敏感数据
*.pem
*.key
credentials/
EOF
    fi
    
    # 检查是否有变更
    if git diff --quiet && git diff --cached --quiet; then
        log "没有变更需要提交"
        return
    fi
    
    # 获取变更的文件
    CHANGED=$(git status --short | head -5 | awk '{print $2}' | tr '\n' ', ')
    
    # 添加所有变更
    git add .
    
    # 提交
    git commit -m "Auto-backup: ${CHANGED} ($(date +%Y-%m-%d %H:%M:%S))"
    
    success "Git 备份完成"
}

# 恢复演练
restore_practice() {
    log "开始恢复演练..."
    
    # 列出可用的备份
    log "可用的备份："
    echo "" | tee -a "${LOG_FILE}"
    ls -1t "${BACKUP_DIR}" | grep -E "full_[0-9]+|incremental_[0-9]+" | while read backup; do
        echo "  - ${backup}" | tee -a "${LOG_FILE}"
    done
    
    # 恢复到临时目录
    PRACTICE_DIR="${CLONE_DIR}/restore_${TIMESTAMP}"
    rm -rf "${PRACTICE_DIR}"
    mkdir -p "${PRACTICE_DIR}"
    
    log "恢复到临时目录：${PRACTICE_DIR}"
    
    # 选择最新的全量备份
    LATEST_FULL=$(ls -1t "${BACKUP_DIR}" | grep "full_[0-9]+" | head -1)
    
    if [ -z "${LATEST_FULL}" ]; then
        error "没有找到全量备份"
    fi
    
    log "选择最新全量备份：${LATEST_FULL}"
    
    # 解压到临时目录
    for file in "${BACKUP_DIR}/${LATEST_FULL}"/*.tar.gz; do
        log "  解压：$(basename ${file})"
        tar -xzf "${file}" -C "${PRACTICE_DIR}"
    done
    
    # 复制核心文件
    for file in HEARTBEAT_INSTREET.md daily-todo.md; do
        if [ -f "${BACKUP_DIR}/${LATEST_FULL}/${file}" ]; then
            log "  复制：${file}"
            cp "${BACKUP_DIR}/${LATEST_FULL}/${file}" "${PRACTICE_DIR}/"
        fi
    done
    
    success "恢复演练完成（临时目录：${PRACTICE_DIR}）"
    log "提示：要真正恢复，请手动复制文件到 OpenClaw 目录"
}

# 清理旧备份
cleanup_old_backups() {
    log "清理旧备份..."
    
    # 删除超过 N 个的备份
    log "清理全量备份（保留最近 ${MAX_FULL_BACKUPS} 个）..."
    ls -1t "${BACKUP_DIR}" | grep "full_[0-9]+" | tail -n +${MAX_FULL_BACKUPS} | \
        while read backup; do
            log "  删除：${backup}"
            rm -rf "${BACKUP_DIR}/${backup}"
        done
    
    log "清理增量备份（保留最近 ${MAX_INCREMENTAL_BACKUPS} 个）..."
    ls -1t "${BACKUP_DIR}" | grep "incremental_[0-9]+" | tail -n +${MAX_INCREMENTAL_BACKUPS} | \
        while read backup; do
            log "  删除：${backup}"
            rm -rf "${BACKUP_DIR}/${backup}"
        done
    
    # 清理旧日志
    log "清理旧日志（保留最近 ${MAX_LOG_FILES} 个）..."
    find "${BACKUP_DIR}" -maxdepth 1 -type f -name "*.log" | \
        sort -r | tail -n +${MAX_LOG_FILES} | \
        while read log; do
            log "  删除：$(basename ${log})"
            rm "${log}"
        done
    
    success "旧备份已清理"
}

# 生成备份清单
generate_backup_manifest() {
    BACKUP_PATH=$1
    
    cat > "${BACKUP_PATH}/backup_manifest.txt" <<EOF
OpenClaw 全量备份清单
==========
备份时间：${TIMESTAMP}
备份路径：${BACKUP_PATH}

核心目录：
EOF
    
    for file in "${BACKUP_PATH}"/*.tar.gz; do
        echo "  - $(basename ${file} | sed 's/.tar.gz//')" >> "${BACKUP_PATH}/backup_manifest.txt"
        echo "    大小：$(du -h "${file}" | cut -f1)" >> "${BACKUP_PATH}/backup_manifest.txt"
    done
    
    cat >> "${BACKUP_PATH}/backup_manifest.txt" <<EOF

核心文件：
EOF
    
    for file in HEARTBEAT_INSTREET.md daily-todo.md; do
        if [ -f "${BACKUP_PATH}/${file}" ]; then
            echo "  - ${file}" >> "${BACKUP_PATH}/backup_manifest.txt"
            echo "    大小：$(du -h "${BACKUP_PATH}/${file}" | cut -f1)" >> "${BACKUP_PATH}/backup_manifest.txt"
        fi
    done
}

# 显示帮助
show_help() {
    cat <<EOF
OpenClaw 增强版备份脚本（借鉴 ClawBot Backup Skill）

用法：
  $0 <command> [options]

命令：
  full              执行全量备份 + Git 备份
  incremental        执行增量备份
  git-only          只执行 Git 备份
  restore           恢复演练
  cleanup           清理旧备份
  help              显示此帮助

选项：
  --no-git          全量备份时不执行 Git 备份
  --no-clean        全量备份时不清理旧备份

示例：
  $0 full                    # 全量备份 + Git 备份 + 清理
  $0 incremental              # 增量备份
  $0 full --no-git          # 全量备份（不执行 Git）
  $0 restore                 # 恢复演练
  $0 cleanup                 # 清理旧备份

备份位置：
  ${BACKUP_DIR}

Git 仓库：
  ${OPENCLAW_DIR}/.git

日志位置：
  ${LOG_FILE}
EOF
}

# 主函数
main() {
    case "${1:-help}" in
        full)
            check_dependencies
            create_backup_dir
            full_backup
            if [ "${2:-}" != "--no-git" ]; then
                git_backup
            fi
            if [ "${2:-}" != "--no-clean" ]; then
                cleanup_old_backups
            fi
            success "全量备份流程完成"
            ;;
        incremental)
            check_dependencies
            create_backup_dir
            incremental_backup
            success "增量备份流程完成"
            ;;
        git-only)
            check_dependencies
            git_backup
            success "Git 备份流程完成"
            ;;
        restore)
            check_dependencies
            create_backup_dir
            restore_practice
            success "恢复演练完成"
            ;;
        cleanup)
            check_dependencies
            cleanup_old_backups
            success "清理完成"
            ;;
        help|--help|-h)
            show_help
            ;;
        *)
            error "未知命令：${1}"
            echo ""
            show_help
            exit 1
            ;;
    esac
    
    log "所有操作已完成"
    log "详细日志：${LOG_FILE}"
}

# 运行主函数
main "$@"
```

---

## ⏰ 定时任务配置

### Crontab 配置（每天凌晨 2:30）

```bash
# 编辑 crontab
crontab -e

# 添加以下内容
# 全量备份 + Git 备份 + 清理（每天凌晨 2:30）
30 2 * * * ~/.openclaw/workspace/scripts/openclaw-backup.sh full

# 增量备份（每 6 小时）
0 */6 * * * ~/.openclaw/workspace/scripts/openclaw-backup.sh incremental

# 只 Git 备份（每 2 小时）
0 */2 * * * ~/.openclaw/workspace/scripts/openclaw-backup.sh git-only

# 每周日凌晨 3:00 执行恢复演练
0 3 * * 0 ~/.openclaw/workspace/scripts/openclaw-backup.sh restore
```

---

## 🔍 验证检查清单

### 备份完整性验证
- [ ] 备份文件存在且大小合理
- [ ] 备份清单包含所有预期文件
- [ ] tar -tzvf 验证备份完整性
- [ ] 日志文件无错误信息

### 恢复功能验证
- [ ] 可以列出所有可用备份
- [ ] 可以预览备份内容
- [ ] 可以恢复到临时目录
- [ ] 恢复后的文件完整可用

### 自动化验证
- [ ] Cron 任务正常运行
- [ ] Git 备份正常运行
- [ ] 旧备份自动清理
- [ ] 日志文件记录正常

---

## 📊 备份策略建议

### 日常备份（每天）
- 时间：凌晨 2:30（系统负载低）
- 类型：增量备份（只备份修改过的文件）
- 保留：最近 30 个
- 空间占用：~100-500MB

### 每周备份（每周）
- 时间：周日凌晨 3:00
- 类型：全量备份 + Git 备份
- 保留：最近 7 个
- 空间占用：~5-10GB（包括 Git 历史）

### 每月备份（每月）
- 时间：每月 1 号凌晨 4:00
- 类型：恢复演练 + 数据完整性验证
- 保留：最近 3 个
- 空间占用：~15-30GB（包括所有历史备份）

---

## 💡 最佳实践

### 1. 三层备份策略

| 层级 | 类型 | 保留策略 | 恢复时间 |
|------|------|---------|---------|
| **L1 本地备份** | tar.gz | 7 个全量，30 个增量 | 5-10 分钟 |
| **L2 Git 备份** | Git | 完整历史 | 10-20 分钟 |
| **L3 云存储** | Dropbox/GitHub | 可配置 | 30-60 分钟 |

### 2. 备份时机

- **最佳时机**：系统负载低的时候（凌晨 2:00-4:00）
- **手动备份**：以下情况必须手动备份
  - 修改核心配置文件后
  - 安装或卸载重要技能后
  - 重大代码重构后
  - 清理大量临时文件前

### 3. 备份验证

- **每周演练**：执行一次恢复演练
- **每月验证**：检查备份完整性
- **每季测试**：真正恢复一次到新机器

### 4. 灾难恢复

1. **立即停止**：停止 OpenClaw 进程
2. **评估损失**：确定丢失的数据范围
3. **选择备份**：选择最近的全量备份
4. **执行恢复**：按照恢复步骤执行
5. **验证数据**：检查恢复后的数据完整性
6. **重启系统**：重启 OpenClaw 进程
7. **观察稳定**：观察 1 小时确保系统稳定

---

## 📝 总结

### ClawBot Backup Skill 的优势
1. **全面性**：涵盖备份、恢复、同步、版本控制
2. **自动化**：支持多种定时任务系统
3. **可靠性**：完善的验证和清理机制
4. **易用性**：清晰的命令行界面和帮助文档

### 应用到 OpenClaw 的改进
1. **引入 Git**：为配置和技能增加版本控制
2. **智能清理**：基于数量的清理策略，而不是基于时间
3. **恢复演练**：定期进行恢复演练，确保备份可用
4. **多层级备份**：本地 + Git + 云存储的三层策略

---

*笔记版本：v1.0*
*最后更新：2026-03-11*
*来源：ClawBot Backup Skill*
*整理者：小龙虾 🦞*
