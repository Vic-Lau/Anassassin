# Anassassin
# Commit-EachFile.ps1
# 逐个提交当前仓库的变更文件；若文件行数 > 2000，则跳过并输出路径与行数。

$ErrorActionPreference = "Stop"

# 切到脚本所在目录，确保在仓库根目录执行
Set-Location -Path $PSScriptRoot

# 基本检查
if (-not (Get-Command git -ErrorAction SilentlyContinue)) {
    Write-Error "未找到 git，请先安装 Git 并确保在 PATH 中。"
    exit 1
}

# 确认在 git 仓库里
try {
    git rev-parse --is-inside-work-tree | Out-Null
} catch {
    Write-Error "当前目录不是 Git 仓库，请在仓库根目录运行。"
    exit 1
}

# 获取“有变更”的文件列表：
# -m: 已修改的已跟踪文件
# -o: 未跟踪(新增)文件（排除忽略）
# --exclude-standard: 按 .gitignore / 标准规则排除忽略文件
$files = git ls-files -m -o --exclude-standard

if ([string]::IsNullOrWhiteSpace($files)) {
    Write-Host "没有需要提交的变更文件。"
    exit 0
}

# 统计行数的函数：尽量省内存、兼容大文件
function Get-LineCount([string]$path) {
    try {
        return [System.IO.File]::ReadLines($path).Count
    } catch {
        # 如果无法按文本读取（例如二进制/编码异常），返回 -1 表示不可测
        return -1
    }
}

# 遍历文件并逐个提交
$skipped = @()
$committed = @()

$files -split "`n" | ForEach-Object {
    $f = $_.Trim()
    if (-not $f) { return }

    # 有些状态下，文件可能被删除或重命名：确保路径存在
    if (-not (Test-Path -LiteralPath $f)) {
        Write-Host "跳过（路径不存在）：$f"
        return
    }

    $lineCount = Get-LineCount -path $f

    if ($lineCount -eq -1) {
        Write-Host "跳过（无法读取为文本，可能为二进制或编码异常）：$f"
        $skipped += [PSCustomObject]@{ File=$f; Reason="非文本/编码异常"; Lines=$null }
        return
    }

    if ($lineCount -gt 2000) {
        Write-Host "跳过（>2000 行）：$f  —— 行数：$lineCount"
        $skipped += [PSCustomObject]@{ File=$f; Reason="行数超限"; Lines=$lineCount }
        return
    }

    # 逐个提交
    Write-Host "提交文件：$f  —— 行数：$lineCount"
    git add -- "$f" | Out-Null

    # 提交信息可按需修改
    $msg = "chore: 单文件提交 $f"
    # --only/-- "$f" 确保只提交该文件
    git commit -m "$msg" -- "$f" | Out-Null

    $committed += [PSCustomObject]@{ File=$f; Lines=$lineCount }
}

Write-Host "`n===== 提交结果汇总 ====="
if ($committed.Count -gt 0) {
    Write-Host "已提交文件：$($committed.Count) 个"
    $committed | ForEach-Object { Write-Host "  ✔ $($_.File)（$($_.Lines) 行）" }
} else {
    Write-Host "没有文件被提交。"
}

if ($skipped.Count -gt 0) {
    Write-Host "`n被跳过文件：$($skipped.Count) 个"
    $skipped | ForEach-Object {
        if ($_.Lines -ne $null) {
            Write-Host "  ✖ $($_.File)（$($_.Lines) 行，原因：$($_.Reason)）"
        } else {
            Write-Host "  ✖ $($_.File)（原因：$($_.Reason)）"
        }
    }
}
