# KaptainKube Dockerfile Deployment

## AI-Driven Container-to-Production Pipeline

### Problem Statement

Modern containerized applications require extensive Kubernetes expertise to deploy securely and efficiently. Developers have a working Dockerfile but need dozens of additional components for production readiness: ingress controllers, service meshes, monitoring, autoscaling, security policies, and CI/CD pipelines. This complexity creates a significant barrier between "it works in Docker" and "it's production-ready on Kubernetes."

### Solution

KaptainKube's Dockerfile deployment capability transforms a simple container definition into a complete, production-grade Kubernetes deployment using AI-driven analysis and the curated component library. 

**Single Command Deployment:**
```bash
kaptain-kube deploy app-config.yaml
```

This command analyzes your Dockerfile and application requirements, then automatically generates and applies all necessary Kubernetes manifests, security policies, monitoring configurations, and CI/CD pipelines.

## Quick Start

### Basic Deployment

**1. Create an application configuration:**
```yaml
# app-config.yaml
apiVersion: kaptainkube.dev/v1
kind: AppDeployment
metadata:
  name: my-web-app
spec:
  dockerfile: ./Dockerfile
  
  requirements:
    - "REST API with database backend"
    - "Handle 1000 concurrent users"
    - "High availability required"
  
  ports:
    - port: 8080
      protocol: HTTP
      path: "/api"
  
  resources:
    cpu: "500m"
    memory: "1Gi"
```

**2. Deploy to Kubernetes:**
```bash
kaptain-kube deploy app-config.yaml
```

**3. KaptainKube automatically provisions:**
- Kubernetes Deployment with optimized resource allocation
- Service and Ingress with Kong API Gateway
- Istio service mesh for mTLS and traffic management
- Prometheus monitoring and Grafana dashboards
- HPA/VPA for auto-scaling
- Security policies via OPA Gatekeeper
- CI/CD pipeline with Tekton and ArgoCD

## Configuration Specification

### AppDeployment Resource Schema

```yaml
apiVersion: kaptainkube.dev/v1
kind: AppDeployment
metadata:
  name: string                    # Application name
  namespace: string               # Target namespace (optional)
  labels: {}                      # Additional labels
spec:
  # Container Source
  dockerfile: string              # Path to Dockerfile (required if image not specified)
  image: string                   # Pre-built container image (alternative to dockerfile)
  buildContext: string            # Docker build context path (default: ".")
  
  # Application Requirements (Natural Language)
  requirements: []string          # High-level functional requirements
  
  # Service Configuration
  ports:
    - port: int                   # Container port
      protocol: string            # HTTP, HTTPS, TCP, UDP
      path: string                # URL path for HTTP services (optional)
      healthCheck: string         # Health check endpoint (optional)
  
  # Resource Specifications
  resources:
    cpu: string                   # CPU request/limit (e.g., "500m")
    memory: string                # Memory request/limit (e.g., "1Gi")
    storage: string               # Persistent storage size (optional)
    gpu: int                      # GPU count (optional)
  
  # Scaling Configuration
  scaling:
    minReplicas: int              # Minimum pod count (default: 1)
    maxReplicas: int              # Maximum pod count (default: 10)
    targetCPU: int                # CPU utilization % for HPA (default: 80)
    targetMemory: int             # Memory utilization % for VPA (optional)
  
  # Security & Compliance
  securityProfile: string         # "standard", "pci", "hipaa", "fips" (default: "standard")
  secretsRequired: []string       # List of secret names needed
  
  # Database Requirements
  database:
    type: string                  # "postgresql", "mysql", "mongodb", "redis"
    size: string                  # Storage size (e.g., "10Gi")
    backup: bool                  # Enable automated backups (default: true)
  
  # Advanced Configuration
  components:
    ingress:
      enabled: bool               # Enable ingress (default: true for HTTP services)
      className: string           # Ingress class (default: "kong")
      tls: bool                   # Enable TLS (default: true)
      annotations: {}             # Custom ingress annotations
    
    serviceMesh:
      enabled: bool               # Enable Istio (default: true)
      mTLS: string                # "strict", "permissive", "disabled" (default: "strict")
    
    monitoring:
      enabled: bool               # Enable observability stack (default: true)
      metrics: bool               # Prometheus metrics (default: true)
      traces: bool                # OpenTelemetry tracing (default: true)
      logs: bool                  # Centralized logging (default: true)
    
    autoscaling:
      enabled: bool               # Enable HPA/VPA (default: true)
      predictive: bool            # Enable predictive scaling (default: false)
    
    cicd:
      enabled: bool               # Enable CI/CD pipeline (default: true)
      gitRepo: string             # Git repository URL
      branch: string              # Target branch (default: "main")
      webhook: bool               # Enable Git webhooks (default: true)
```

## Architecture Integration

### Component Selection Matrix

KaptainKube automatically selects and configures components based on Dockerfile analysis and requirements:

| Dockerfile Analysis | Auto-Selected Components |
|---------------------|--------------------------|
| `EXPOSE 80/443` | Kong Ingress + cert-manager |
| `FROM postgres:*` | Skip database operator |
| `ENV DATABASE_URL` | Zalando Postgres Operator |
| `USER 1000:1000` | Restrictive security policies |
| `HEALTHCHECK` instruction | Kubernetes probes auto-configured |
| Multi-stage build | Optimize resource allocation |
| `COPY requirements.txt` | Language-specific monitoring |

### AI Analysis Engine

The deployment process uses LLM analysis to:

1. **Parse Dockerfile**: Extract runtime requirements, dependencies, and security context
2. **Analyze Requirements**: Map natural language requirements to technical components
3. **Resource Estimation**: Predict CPU/memory needs based on application type
4. **Security Assessment**: Determine appropriate security policies and compliance requirements
5. **Architecture Planning**: Select optimal component configuration

### Generated Kubernetes Resources

For each deployment, KaptainKube generates:

```
📁 k8s-manifests/
├── 01-namespace.yaml              # Namespace with labels and policies
├── 02-secrets.yaml                # Auto-generated secrets and certificates
├── 03-configmap.yaml              # Application configuration
├── 04-deployment.yaml             # Main application deployment
├── 05-service.yaml                # ClusterIP service
├── 06-ingress.yaml                # Kong ingress with TLS
├── 07-hpa.yaml                     # Horizontal Pod Autoscaler
├── 08-vpa.yaml                     # Vertical Pod Autoscaler
├── 09-servicemonitor.yaml          # Prometheus monitoring
├── 10-networkpolicy.yaml           # Network security policies
├── 11-podsecuritypolicy.yaml       # Pod security standards
├── 12-istio-virtualservice.yaml    # Service mesh configuration
├── 13-istio-destinationrule.yaml   # Traffic policies
└── 14-argo-rollout.yaml            # Progressive delivery configuration

📁 ci-cd/
├── tekton-pipeline.yaml            # Build and test pipeline
├── tekton-trigger.yaml             # Git webhook trigger
└── argocd-application.yaml         # GitOps deployment

📁 monitoring/
├── grafana-dashboard.yaml          # Application-specific dashboard
├── prometheus-rules.yaml           # Alerting rules
└── jaeger-tracing.yaml             # Distributed tracing config
```

## Implementation Requirements

### Phase 1: Core Implementation

#### 1.1 CLI Command Structure
```go
// Add to cmd/main.go
rootCmd.AddCommand(&cobra.Command{
    Use:   "deploy [config-file]",
    Short: "Deploy containerized application with production-grade Kubernetes setup",
    Long: `Analyze Dockerfile and requirements to generate complete Kubernetes deployment
    including ingress, monitoring, autoscaling, security policies, and CI/CD pipeline.`,
    Args:  cobra.ExactArgs(1),
    RunE: func(cmd *cobra.Command, args []string) error {
        return RunDeployCommand(cmd.Context(), *opt, args[0])
    },
})
```

#### 1.2 Configuration Parser
```go
// pkg/deployment/config.go
type AppDeploymentConfig struct {
    APIVersion string                 `yaml:"apiVersion"`
    Kind       string                 `yaml:"kind"`
    Metadata   AppMetadata            `yaml:"metadata"`
    Spec       AppDeploymentSpec      `yaml:"spec"`
}

type AppDeploymentSpec struct {
    Dockerfile       string              `yaml:"dockerfile,omitempty"`
    Image           string              `yaml:"image,omitempty"`
    Requirements    []string            `yaml:"requirements,omitempty"`
    Ports           []ServicePort       `yaml:"ports,omitempty"`
    Resources       ResourceConfig      `yaml:"resources,omitempty"`
    // ... other fields
}
```

#### 1.3 Dockerfile Analyzer
```go
// pkg/deployment/dockerfile.go
type DockerfileAnalyzer struct {
    LLMClient gollm.Client
}

type DockerfileAnalysis struct {
    BaseImage       string
    ExposedPorts    []int
    User            string
    WorkDir         string
    Dependencies    []string
    HealthCheck     *HealthCheckConfig
    ResourceHints   ResourceEstimate
    SecurityContext SecurityAnalysis
}

func (da *DockerfileAnalyzer) Analyze(ctx context.Context, dockerfilePath string) (*DockerfileAnalysis, error)
```

#### 1.4 Component Template Engine
```go
// pkg/deployment/templates.go
type TemplateEngine struct {
    ComponentLibrary map[string]ComponentTemplate
    LLMClient       gollm.Client
}

type ComponentTemplate struct {
    Name         string
    Type         ComponentType // Ingress, ServiceMesh, Database, etc.
    Dependencies []string
    Manifests    []string // YAML template files
    Configurator func(AppDeploymentSpec, DockerfileAnalysis) (map[string]interface{}, error)
}

func (te *TemplateEngine) GenerateManifests(spec AppDeploymentSpec, analysis DockerfileAnalysis) ([]KubernetesManifest, error)
```

#### 1.5 LLM Integration
```go
// pkg/deployment/planner.go
type DeploymentPlanner struct {
    LLMClient gollm.Client
    Templates *TemplateEngine
}

type DeploymentPlan struct {
    Components      []ComponentConfig
    Dependencies    []DependencySpec
    SecurityPolicies []SecurityPolicy
    MonitoringRules []MonitoringConfig
    ScalingConfig   AutoscalingConfig
}

func (dp *DeploymentPlanner) CreatePlan(ctx context.Context, config AppDeploymentConfig, analysis DockerfileAnalysis) (*DeploymentPlan, error)
```

### Phase 2: Component Library Integration

#### 2.1 Component Templates Directory Structure
```
📁 pkg/deployment/templates/
├── ingress/
│   ├── kong/
│   │   ├── ingress.yaml.tmpl
│   │   ├── kong-plugin.yaml.tmpl
│   │   └── certificate.yaml.tmpl
│   └── nginx/
├── servicemesh/
│   ├── istio/
│   │   ├── virtualservice.yaml.tmpl
│   │   ├── destinationrule.yaml.tmpl
│   │   └── peerauthentication.yaml.tmpl
├── database/
│   ├── postgresql/
│   │   ├── cluster.yaml.tmpl
│   │   ├── backup.yaml.tmpl
│   │   └── monitoring.yaml.tmpl
├── monitoring/
│   ├── prometheus/
│   └── grafana/
├── autoscaling/
│   ├── hpa.yaml.tmpl
│   ├── vpa.yaml.tmpl
│   └── keda.yaml.tmpl
└── security/
    ├── networkpolicy.yaml.tmpl
    ├── podsecuritypolicy.yaml.tmpl
    └── opa-policies/
```

#### 2.2 Component Registry
```go
// pkg/deployment/registry.go
type ComponentRegistry struct {
    components map[string]Component
    resolver   DependencyResolver
}

type Component interface {
    Name() string
    Type() ComponentType
    Dependencies() []string
    ApplicabilityRules() []Rule
    Generate(config Config) ([]KubernetesResource, error)
    Validate(config Config) error
}

// Built-in components following the white paper's "Lego" library
func RegisterBuiltinComponents(registry *ComponentRegistry) {
    registry.Register(NewKongIngressComponent())
    registry.Register(NewIstioServiceMeshComponent())
    registry.Register(NewPrometheusMonitoringComponent())
    registry.Register(NewPostgresOperatorComponent())
    registry.Register(NewHPAComponent())
    registry.Register(NewOPAGatekeeperComponent())
    // ... etc.
}
```

### Phase 3: Advanced Features

#### 3.1 GitOps Integration
```go
// pkg/deployment/gitops.go
type GitOpsManager struct {
    GitClient    GitClient
    ArgoClient   ArgoClient
    TemplateRepo string
}

func (gm *GitOpsManager) CommitManifests(manifests []KubernetesManifest, config AppDeploymentConfig) error
func (gm *GitOpsManager) CreateArgoApplication(config AppDeploymentConfig) error
func (gm *GitOpsManager) SetupWebhooks(repoURL string) error
```

#### 3.2 CI/CD Pipeline Generation
```go
// pkg/deployment/cicd.go
type PipelineGenerator struct {
    TektonClient tekton.Interface
    Templates    map[string]PipelineTemplate
}

type PipelineTemplate struct {
    BuildSteps    []tektonv1.Step
    TestSteps     []tektonv1.Step
    SecurityScans []tektonv1.Step
    DeploySteps   []tektonv1.Step
}

func (pg *PipelineGenerator) GeneratePipeline(config AppDeploymentConfig, analysis DockerfileAnalysis) (*tektonv1.Pipeline, error)
```

#### 3.3 Monitoring & Observability
```go
// pkg/deployment/observability.go
type ObservabilityStack struct {
    PrometheusConfig *PrometheusConfig
    GrafanaConfig    *GrafanaConfig
    JaegerConfig     *JaegerConfig
    FluentBitConfig  *FluentBitConfig
}

func (os *ObservabilityStack) GenerateDashboard(appMetadata AppMetadata, ports []ServicePort) (*GrafanaDashboard, error)
func (os *ObservabilityStack) GenerateAlertRules(slos []SLOSpec) (*PrometheusRules, error)
```

### Phase 4: Testing & Validation

#### 4.1 Integration Tests
```go
// test/integration/deployment_test.go
func TestDockerfileDeployment(t *testing.T) {
    // Test cases for different Dockerfile patterns
    testCases := []struct {
        name           string
        dockerfile     string
        config         AppDeploymentConfig
        expectedComps  []string
        expectedPolicies []string
    }{
        {
            name: "simple web app",
            dockerfile: "testdata/simple-web-app/Dockerfile",
            config: AppDeploymentConfig{...},
            expectedComps: []string{"kong-ingress", "istio-mesh", "prometheus"},
        },
        // ... more test cases
    }
}
```

#### 4.2 Component Validation
```go
// pkg/deployment/validator.go
type ManifestValidator struct {
    KubeClient kubernetes.Interface
    LintRules  []LintRule
}

func (mv *ManifestValidator) ValidateManifests(manifests []KubernetesManifest) ([]ValidationError, error)
func (mv *ManifestValidator) DryRunDeploy(manifests []KubernetesManifest) error
```

## Usage Examples

### Simple Web Application

```yaml
# web-app.yaml
apiVersion: kaptainkube.dev/v1
kind: AppDeployment
metadata:
  name: blog-api
spec:
  dockerfile: ./Dockerfile
  requirements:
    - "REST API for blog platform"
    - "Serves 10,000 daily users"
    - "PostgreSQL database required"
  
  ports:
    - port: 3000
      protocol: HTTP
      path: "/api"
      healthCheck: "/health"
  
  database:
    type: postgresql
    size: "20Gi"
```

**Generated deployment includes:**
- Node.js application deployment with resource optimization
- PostgreSQL cluster with backup configuration
- Kong ingress with rate limiting
- Istio service mesh with circuit breakers
- Prometheus monitoring with custom dashboards
- HPA scaling based on request rate

### Microservice with gRPC

```yaml
# user-service.yaml
apiVersion: kaptainkube.dev/v1
kind: AppDeployment
metadata:
  name: user-service
spec:
  dockerfile: ./Dockerfile
  requirements:
    - "gRPC user management service"
    - "PCI compliance required"
    - "High availability with 99.9% uptime"
  
  ports:
    - port: 9090
      protocol: gRPC
  
  securityProfile: "pci"
  
  scaling:
    minReplicas: 3
    maxReplicas: 50
    targetCPU: 70
```

### Machine Learning Inference

```yaml
# ml-inference.yaml
apiVersion: kaptainkube.dev/v1
kind: AppDeployment
metadata:
  name: image-classifier
spec:
  dockerfile: ./Dockerfile
  requirements:
    - "TensorFlow image classification"
    - "GPU acceleration required"
    - "Model serving with batch processing"
  
  resources:
    gpu: 1
    memory: "8Gi"
  
  scaling:
    minReplicas: 1
    maxReplicas: 5
    predictive: true
```

## Error Handling & Troubleshooting

### Common Issues

**Dockerfile Analysis Failures:**
```bash
# Enable verbose analysis
kaptain-kube deploy app-config.yaml --analyze-verbose

# Manual Dockerfile validation
kaptain-kube validate-dockerfile ./Dockerfile
```

**Component Conflicts:**
```bash
# Show selected components and dependencies
kaptain-kube deploy app-config.yaml --dry-run --show-components

# Override component selection
kaptain-kube deploy app-config.yaml --disable-component=istio
```

**Resource Estimation Issues:**
```bash
# Override resource estimates
kaptain-kube deploy app-config.yaml --resources-cpu=1000m --resources-memory=2Gi
```

### Validation Commands

```bash
# Validate configuration before deployment
kaptain-kube validate app-config.yaml

# Generate manifests without applying
kaptain-kube deploy app-config.yaml --generate-only --output-dir=./k8s

# Show deployment plan
kaptain-kube deploy app-config.yaml --plan-only
```

## Future Enhancements

### Phase 5: Advanced AI Features

- **Auto-optimization**: Continuous resource and configuration optimization based on runtime metrics
- **Predictive scaling**: ML-based traffic prediction for proactive scaling
- **Security recommendations**: AI-driven security policy suggestions
- **Cost optimization**: Automated cost analysis and optimization recommendations

### Phase 6: Enterprise Features

- **Multi-cluster deployment**: Deploy across multiple Kubernetes clusters
- **Compliance automation**: Automated compliance reporting and policy enforcement
- **Disaster recovery**: Automated DR planning and testing
- **Advanced observability**: Custom SLI/SLO generation and monitoring

## Benefits

- **Developer Productivity**: Transform Dockerfile to production in minutes, not weeks
- **Production Readiness**: Automatic security, monitoring, and scaling configuration
- **Best Practices**: Enforced Kubernetes and cloud-native best practices
- **Consistency**: Standardized deployment patterns across teams
- **Reduced Complexity**: Abstract away Kubernetes complexity while maintaining flexibility
- **Cost Optimization**: Built-in resource optimization and scaling policies

This capability positions KaptainKube as the bridge between containerized applications and production-ready Kubernetes deployments, embodying the "describe the app; let the agent build the platform" vision. 