直接给权限运行即可
```sh
#!/bin/bash

# 配置信息
ACCESS_KEY_ID="" # 您的AccessKeyId
ACCESS_KEY_SECRET=""   # 请替换为您的密钥
DOMAIN=""
RR=""  # 子域名前缀
TYPE="A"

# 获取当前IP
CURRENT_IP=$(curl -s http://ipv4.icanhazip.com)
echo "当前IP: $CURRENT_IP"

# 标准URL编码函数
urlencode() {
    local length="${#1}"
    local i char
    for (( i = 0; i < length; i++ )); do
        char="${1:i:1}"
        case $char in
            [a-zA-Z0-9.~_-]) printf "$char" ;;
            *) printf '%%%02X' "'$char" ;;
        esac
    done
}

# 发送请求
send_request() {
    local action="$1"
    local params="$2"
    local timestamp=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
    local nonce=$(cat /proc/sys/kernel/random/uuid 2>/dev/null || date +%s%N | md5sum | cut -d ' ' -f 1)
    
    # 1. 构建公共参数字符串
    local common_params="AccessKeyId=${ACCESS_KEY_ID}&Action=${action}&Format=JSON&SignatureMethod=HMAC-SHA1&SignatureNonce=${nonce}&SignatureVersion=1.0&Timestamp=$(urlencode "$timestamp")&Version=2015-01-09"
    
    # 2. 添加方法特定参数
    local full_params="$common_params"
    if [ -n "$params" ]; then
        full_params="${full_params}&${params}"
    fi
    
    # 3. 对参数进行排序
    local sorted_params=$(echo "$full_params" | tr '&' '\n' | sort | tr '\n' '&' | sed 's/&$//')
    
    # 4. 构建签名
    local string_to_sign="GET&%2F&$(urlencode "$sorted_params" | sed 's/%3A/%253A/g')"
    
    # 调试信息
    echo "待签名字符串: $string_to_sign"
    
    # 5. 计算签名
    local key="${ACCESS_KEY_SECRET}&"
    local signature=$(echo -n "$string_to_sign" | openssl dgst -sha1 -hmac "$key" -binary | base64)
    
    # 6. 构建最终URL
    local url="https://alidns.aliyuncs.com/?${sorted_params}&Signature=$(urlencode "$signature")"
    
    # 调试信息
    echo "请求URL: $url"
    
    # 7. 发送请求
    local response=$(curl -s "$url")
    echo "$response"
}

# 查询域名记录
echo "查询域名记录..."
response=$(send_request "DescribeDomainRecords" "DomainName=${DOMAIN}&RRKeyWord=${RR}")
echo "查询响应: $response"

# 检查是否有错误
if echo "$response" | grep -q "\"Code\":"; then
    error_code=$(echo "$response" | grep -o '"Code":"[^"]*"' | cut -d'"' -f4)
    error_message=$(echo "$response" | grep -o '"Message":"[^"]*"' | cut -d'"' -f4)
    echo "错误: $error_code - $error_message"
    exit 1
fi

# 提取记录ID
record_id=$(echo "$response" | grep -o '"RecordId":"[^"]*"' | head -1 | cut -d'"' -f4)

if [ -z "$record_id" ]; then
    echo "未找到记录ID，尝试创建新记录..."
    # 创建新记录
    response=$(send_request "AddDomainRecord" "DomainName=${DOMAIN}&RR=${RR}&Type=${TYPE}&Value=${CURRENT_IP}")
    echo "创建响应: $response"
    if echo "$response" | grep -q "RecordId"; then
        echo "成功创建记录"
        exit 0
    else
        echo "创建记录失败"
        exit 1
    fi
else
    echo "找到记录ID: $record_id"
    
    # 获取当前记录值
    current_value=$(echo "$response" | grep -o '"Value":"[^"]*"' | head -1 | cut -d'"' -f4)
    echo "当前记录值: $current_value"
    
    if [ "$current_value" = "$CURRENT_IP" ]; then
        echo "IP没有变化，无需更新"
        exit 0
    fi
    
    # 更新记录
    echo "更新记录..."
    response=$(send_request "UpdateDomainRecord" "RecordId=${record_id}&RR=${RR}&Type=${TYPE}&Value=${CURRENT_IP}")
    echo "更新响应: $response"
    
    if echo "$response" | grep -q "RecordId"; then
        echo "成功更新记录"
        exit 0
    else
        echo "更新记录失败"
        exit 1
    fi
fi

```

下面是开机自动运行的脚本

```sh
#!/bin/bash

# 检查root权限
if [ "$(id -u)" -ne 0 ]; then
    echo "请使用sudo运行此脚本"
    exit 1
fi

# 确认ddns.sh脚本存在
if [ ! -f "./ddns.sh" ]; then
    echo "错误：当前目录未找到ddns.sh脚本"
    exit 1
fi

# 复制脚本到系统目录
cp ./ddns.sh /usr/local/bin/ddns.sh
chmod +x /usr/local/bin/ddns.sh
echo "DDNS脚本已安装到系统目录"

# 创建日志文件
touch /var/log/ddns.log
chmod 666 /var/log/ddns.log
echo "日志文件已创建"

# 询问用户选择哪种方法
echo "请选择自动运行方式："
echo "1) 使用Crontab (推荐)"
echo "2) 使用Systemd服务和定时器"
read -p "请输入选择 (1/2): " choice

case $choice in
    1)
        # 使用crontab
        (crontab -l 2>/dev/null; echo "# 每10分钟更新一次DDNS") | crontab -
        (crontab -l 2>/dev/null; echo "*/10 * * * * /usr/local/bin/ddns.sh >> /var/log/ddns.log 2>&1") | crontab -
        (crontab -l 2>/dev/null; echo "# 系统重启后也运行一次") | crontab -
        (crontab -l 2>/dev/null; echo "@reboot /usr/local/bin/ddns.sh >> /var/log/ddns.log 2>&1") | crontab -
        echo "Crontab配置完成"
        echo "您的DDNS脚本将每10分钟运行一次，并在系统重启后自动启动"
        ;;
    2)
        # 使用systemd
        cat > /etc/systemd/system/ddns.service << EOF
[Unit]
Description=Aliyun DDNS Update Service
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/ddns.sh
StandardOutput=append:/var/log/ddns.log
StandardError=append:/var/log/ddns.log

[Install]
WantedBy=multi-user.target
EOF

        cat > /etc/systemd/system/ddns.timer << EOF
[Unit]
Description=Run Aliyun DDNS Update every 10 minutes
Requires=ddns.service

[Timer]
Unit=ddns.service
OnBootSec=1min
OnUnitActiveSec=10min
AccuracySec=1s

[Install]
WantedBy=timers.target
EOF

        systemctl daemon-reload
        systemctl enable ddns.timer
        systemctl start ddns.timer
        echo "Systemd定时器配置完成"
        echo "您的DDNS脚本将每10分钟运行一次，并在系统重启后自动启动"
        systemctl status ddns.timer
        ;;
    *)
        echo "无效的选择，退出"
        exit 1
        ;;
esac

echo "==============================="
echo "配置已完成！您可以通过以下命令查看日志："
echo "tail -f /var/log/ddns.log"
echo "==============================="

# 最后立即运行一次脚本
echo "立即运行一次DDNS更新..."
/usr/local/bin/ddns.sh

```

1. 将上面的脚本内容保存为`setup_ddns_auto.sh`
2. 赋予执行权限：`chmod +x setup_ddns_auto.sh`
3. 使用sudo运行：`sudo ./setup_ddns_auto.sh`
4. `tail -f /var/log/ddns.log`查看日志
