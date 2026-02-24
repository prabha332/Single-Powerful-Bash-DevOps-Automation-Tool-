# Single-Powerful-Bash-DevOps-Automation-Tool

#!/bin/bash

# ===============================
# DevOpsCTL - All-in-One Tool
# ===============================

LOG_FILE="./logs/devopsctl.log"
NAMESPACE="default"

mkdir -p logs

log() {
    echo "$(date) - $1" | tee -a $LOG_FILE
}

show_help() {
    echo "Usage: ./devopsctl.sh [command]"
    echo ""
    echo "Commands:"
    echo "  health        Check Kubernetes pod health"
    echo "  auto-heal     Restart failed pods"
    echo "  scale         Scale deployment"
    echo "  backup        Backup PostgreSQL DB"
    echo "  cpu-check     Check server CPU usage"
    echo "  dr-test       Simulate disaster recovery"
    echo "  help          Show this help"
}

check_health() {
    log "Checking pod health..."
    kubectl get pods -n $NAMESPACE
}

auto_heal() {
    log "Auto-healing failed pods..."

    FAILED=$(kubectl get pods -n $NAMESPACE --no-headers | \
    grep -E 'CrashLoopBackOff|Error|ImagePullBackOff' | awk '{print $1}')

    for pod in $FAILED; do
        log "Restarting pod: $pod"
        kubectl delete pod $pod -n $NAMESPACE
    done
}

scale_deployment() {
    read -p "Enter deployment name: " DEPLOY
    read -p "Enter replica count: " REPLICAS

    kubectl scale deployment $DEPLOY --replicas=$REPLICAS -n $NAMESPACE
    log "Scaled $DEPLOY to $REPLICAS replicas"
}

backup_db() {
    DB_NAME="mydb"
    BACKUP_FILE="backup_$(date +%F).sql"

    log "Backing up database..."
    pg_dump $DB_NAME > $BACKUP_FILE
    log "Backup saved as $BACKUP_FILE"
}

cpu_check() {
    CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}')
    log "Current CPU usage: $CPU%"
}

dr_test() {
    log "Simulating Disaster Recovery..."
    echo "Re-deploying infrastructure..."
    # terraform apply -auto-approve
    log "DR simulation complete"
}

# ===============================
# Command Switch
# ===============================

case "$1" in
    health)
        check_health
        ;;
    auto-heal)
        auto_heal
        ;;
    scale)
        scale_deployment
        ;;
    backup)
        backup_db
        ;;
    cpu-check)
        cpu_check
        ;;
    dr-test)
        dr_test
        ;;
    help)
        show_help
        ;;
    *)
        show_help
        ;;
esac
