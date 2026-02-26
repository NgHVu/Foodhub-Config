# Kho lưu trữ cấu hình GitOps FoodHub

![ArgoCD](https://img.shields.io/badge/ArgoCD-ef7b4d?style=for-the-badge&logo=argo&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white)
![Kubernetes](https://img.shields.io/badge/kubernetes-326ce5.svg?&style=for-the-badge&logo=kubernetes&logoColor=white)

Kho lưu trữ các cấu hình triển khai của ứng dụng FoodHub Microservices. Nó được vận hành tự động bởi **Argo CD** theo các nguyên tắc chuẩn của GitOps.

## 🏗 Kiến trúc & Phương pháp tiếp cận

Kho lưu trữ này sử dụng mô hình **App of Apps** kết hợp với **Helm Umbrella Chart** để quản lý việc triển khai đa môi trường (Dev, Staging, Production) trên AWS EKS một cách bảo mật và hiệu quả.

### Các thành phần chính

1. **`helm/foodhub-chart/` (Template Cơ sở):**
   * Một Helm chart tùy chỉnh, có khả năng tái sử dụng, được thiết kế riêng cho kiến trúc microservices.
   * Tích hợp **Argo Rollouts** để hỗ trợ các chiến lược triển khai nâng cao (như triển khai Blue/Green).
   * Định nghĩa các tài nguyên Kubernetes tiêu chuẩn (Services, Ingress, HPA, và cấu hình bảo mật Pod).

2. **`environments/` (Cấu hình riêng theo môi trường):**
   * Chứa các file `values.yaml` cụ thể cho từng microservice (Users, Products, Orders, Frontend).
   * Ghi đè (Override) các giá trị Helm mặc định để tiêm (inject) các cấu hình đặc thù của môi trường (ví dụ: giới hạn tài nguyên, số lượng bản sao, image tags).
   * Chứa thư mục `base-config/` quản lý các ConfigMap và Secret dùng chung cho từng môi trường cụ thể.

3. **`argocd-apps/` (Mô hình App of Apps):**
   * Các file manifest khai báo Ứng dụng (Application) của Argo CD.
   * Kết nối các Helm template với các file cấu hình môi trường và triển khai chúng vào đúng namespace trên Kubernetes (ví dụ: `foodhub-dev`, `foodhub-prod`).

## 🔄 Quy trình hoạt động GitOps

Kho lưu trữ này được tích hợp chặt chẽ với quy trình CI pipeline (Jenkins):

1. **Tích hợp liên tục (CI):** Sau khi Jenkins hoàn tất quá trình build, quét bảo mật (SonarQube/Trivy), và đẩy một Docker image mới lên AWS ECR thành công, nó sẽ tự động commit `image.tag` mới vào file `values.yaml` của môi trường tương ứng trong kho lưu trữ này.
2. **Triển khai liên tục (CD):** Argo CD liên tục giám sát kho lưu trữ này và phát hiện ra sự thay đổi cấu hình (trạng thái OutOfSync).
3. **Đồng bộ hóa tự động:** Argo CD tự động áp dụng các manifest đã được cập nhật xuống cluster AWS EKS, kích hoạt quá trình triển khai (rollout) phiên bản mới một cách an toàn.
4. **Tự phục hồi (Self-Healing):** Nếu có bất kỳ sự thay đổi thủ công, trái phép nào xảy ra trực tiếp trên cluster, Argo CD sẽ tự động ghi đè chúng bằng trạng thái đã được định nghĩa trong kho lưu trữ này, đảm bảo tính tuân thủ tuyệt đối.

## 📂 Cấu trúc Thư mục

```text
.
├── argocd-apps/             # Khai báo Argo CD Application, ánh xạ code vào cluster
│   ├── dev.yaml
│   ├── prod.yaml
│   └── staging.yaml
├── environments/            # Các file cấu hình cơ sở và values riêng cho từng môi trường
│   ├── dev/
│   ├── prod/
│   └── staging/
└── helm/                    # Các Helm charts tái sử dụng cho mọi microservices
    └── foodhub-chart/
        ├── templates/       # Template của Kubernetes manifest (Rollouts, Services, v.v.)
        ├── Chart.yaml
        └── values.yaml
