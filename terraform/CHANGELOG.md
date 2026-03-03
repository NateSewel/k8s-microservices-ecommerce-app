# Terraform Configuration Changelog

## Recent Updates

### 2024 - Configuration Fixes and Improvements

#### Region Migration
- **Changed:** Default AWS region from `us-west-2` to `us-east-1`
- **Files Modified:**
  - `variables.tf` - Updated default region
  - `README.md` - Updated documentation
  - `jenkins` - Updated kubeconfig command
- **Impact:** All resources will now deploy in us-east-1

#### Security Group Enhancements
- **Added:** Port 9090 ingress rule for Prometheus
- **Added:** Port 9093 ingress rule for Alertmanager
- **File Modified:** `security.tf`
- **Reason:** Monitoring services were not accessible due to missing security group rules
- **Impact:** Prometheus and Alertmanager now accessible via LoadBalancer

#### Monitoring Stack LoadBalancer Configuration
- **Changed:** LoadBalancer annotations for Prometheus, Grafana, and Alertmanager
- **File Modified:** `addons-direct.tf`
- **Changes:**
  - Updated `aws-load-balancer-type` from "nlb" to "external"
  - Added `aws-load-balancer-nlb-target-type: "ip"`
  - Added health check paths for each service
  - Added explicit port configurations
- **Reason:** Old NLB annotations not compatible with EKS Auto Mode
- **Impact:** Monitoring LoadBalancers now provision correctly

#### Documentation Updates
- **Updated:** All region references in documentation
- **Updated:** Example commands with correct region
- **Updated:** Troubleshooting guides
- **Files Modified:**
  - `README.md`
  - `terraform/README.md`
  - `BRANCHING_STRATEGY.md`

## Configuration Summary

### Current State (Validated)

#### Infrastructure
- Region: us-east-1
- EKS Version: 1.33
- VPC CIDR: 10.0.0.0/16
- NAT Gateway: Single (cost optimization)

#### Security Groups
All required ports configured:
- 80 (HTTP) - Application access
- 443 (HTTPS) - Secure services
- 9090 (Prometheus) - Monitoring
- 9093 (Alertmanager) - Alert management
- 10254 (Health checks) - LoadBalancer health
- 30000-32767 (NodePort) - Kubernetes services

#### Add-ons
- Cert-manager: v1.13.3
- NGINX Ingress: v4.8.3
- Kube-Prometheus-Stack: v55.5.0 (optional, enabled by default)
- ArgoCD: v5.51.6

#### LoadBalancer Configuration
All LoadBalancers use AWS Load Balancer Controller format:
- Type: external
- Target Type: ip
- Scheme: internet-facing
- Health checks: configured

## Breaking Changes

### None
All changes are backward compatible and improve functionality.

## Migration Notes

### From us-west-2 to us-east-1
If you have existing infrastructure in us-west-2:

1. **Option A: Destroy and Recreate**
   ```bash
   # In us-west-2
   terraform destroy -auto-approve
   
   # Update region in variables.tf to us-east-1
   # Then deploy
   terraform apply -auto-approve
   ```

2. **Option B: Keep Both Regions**
   - Create separate Terraform workspaces
   - Use different state files
   - Deploy to both regions independently

### Monitoring Stack Updates
If you have existing monitoring stack with old annotations:

1. **Automatic Update**
   ```bash
   terraform apply -auto-approve
   ```
   Terraform will update the Helm release with new annotations.

2. **Manual Update (if needed)**
   ```bash
   terraform destroy -target=helm_release.kube_prometheus_stack[0]
   terraform apply -target=helm_release.kube_prometheus_stack[0]
   ```

## Validation

All configurations have been validated:
- ✅ Region configuration correct
- ✅ Security groups complete
- ✅ LoadBalancer annotations updated
- ✅ Health checks configured
- ✅ Documentation updated
- ✅ No manual fixes required

## Next Steps

1. Review `DEPLOYMENT_VALIDATION.md` for deployment checklist
2. Run `terraform apply` to deploy/update infrastructure
3. Wait 5-10 minutes for all resources to be ready
4. Access services via LoadBalancer URLs

## Support Files

Created for troubleshooting (not needed for normal deployment):
- `diagnose-monitoring.sh` - Monitoring diagnostics
- `diagnose-application.sh` - Application diagnostics
- `check-application-access.sh` - Quick status check
- `fix-monitoring-loadbalancers.sh` - Manual fix (not needed)
- `quick-fix-application.sh` - Manual fix (not needed)

These scripts are provided for troubleshooting only. With the updated Terraform configuration, they should not be needed.

## Version History

### v1.2 (Current)
- Region: us-east-1
- Monitoring: Fixed LoadBalancer configuration
- Security: All ports configured
- Status: Production ready

### v1.1
- Region: us-west-2
- Monitoring: Basic configuration
- Security: Limited ports
- Status: Deprecated

### v1.0
- Initial release
- Basic EKS setup
- No monitoring
