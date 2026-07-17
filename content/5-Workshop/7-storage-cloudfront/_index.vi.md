---
title : "Lưu trữ & CloudFront (làm thêm)"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 7. </b> "
---

#### Tổng quan

Đây là chương **làm thêm** — một phần tùy chọn, đào sâu hơn. Ở Chương 5, stack SAM đã tạo một bucket lưu trữ asset riêng tư (`lingorise-assets-dev-<accountId>`) cho các tài nguyên học tập của LingoRise: các đoạn audio, hình ảnh câu hỏi, và tệp học viên tải lên. Trong môi trường `dev`, backend phục vụ các asset này trực tiếp từ S3 và mọi thứ vẫn chạy tốt. Vậy tại sao lại thêm CloudFront?

Có ba lý do. **Độ trễ** — CloudFront cache asset tại các edge location gần học viên, nên một đoạn audio bài nghe sẽ tải nhanh dù người dùng ở Hà Nội hay Thành phố Hồ Chí Minh. **Truy cập có chữ ký** — bạn có thể giữ bucket hoàn toàn riêng tư mà vẫn phát ra các URL có chữ ký, tồn tại ngắn hạn, để chỉ học viên đã đăng nhập và được cấp quyền mới lấy được asset. **Giảm tải cho Lambda** — một khi URL đã được ký thì nó cố định trong suốt TTL, nên các lần lấy lặp lại sẽ trúng cache CloudFront thay vì phải đi vòng qua API và S3 ở mỗi request.

Backend hỗ trợ ba chế độ URL asset, chọn qua biến môi trường `ASSET_URL_MODE`:

| Chế độ | Độ trễ | Chi phí | Thiết lập |
|---|---|---|---|
| `public` | Trực tiếp `storage_url` | S3 GET | Bucket công khai |
| `signed` | Ký một lần mỗi phản hồi | S3 GET | Chỉ cần thông tin IAM |
| `cloudfront_signed` | Không (URL cố định trong TTL) | Cache CloudFront | Cặp khóa + distribution |

{{% notice note %}}
Chương này là **tùy chọn**. Ứng dụng LingoRise chạy hoàn toàn tốt trong `dev` với asset phục vụ trực tiếp từ S3 (`ASSET_URL_MODE=public` hoặc `signed`). Chỉ thêm CloudFront khi bạn muốn cache tại edge và phân phối asset riêng tư có chữ ký qua CDN. Bạn có thể chuyển thẳng sang Chương 8 và quay lại phần này sau.
{{% /notice %}}

#### Bucket lưu trữ asset

Nhớ lại **gap #3** từ Chương 5: template SAM đã tạo một bucket S3 riêng tư tên `lingorise-assets-dev-<accountId>`. Nó được khóa chặt — mọi quyền truy cập công khai đều bị chặn — và IAM role thực thi của Lambda chỉ được cấp vừa đủ quyền S3 để quản lý asset.

Nếu bạn cần tự tạo bucket bằng tay (ví dụ một bucket `prod` riêng), mẫu lệnh như sau:

```bash
aws s3api create-bucket \
  --bucket lingorise-assets-dev-<accountId> \
  --region ap-southeast-1 \
  --create-bucket-configuration LocationConstraint=ap-southeast-1

aws s3api put-public-access-block \
  --bucket lingorise-assets-dev-<accountId> \
  --public-access-block-configuration BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
```

Các quyền IAM backend cần trên bucket này:

- `s3:PutObject`, `s3:DeleteObject` — tải lên và dọn dẹp asset.
- `s3:ListBucket` — dùng cho tác vụ dọn dẹp.
- `s3:GetObject` — chỉ khi `ASSET_URL_MODE=signed` (backend tự pre-sign URL của S3).

Storage driver của backend được cấu hình qua các biến môi trường này (đã có trong SSM tại `/lingorise/dev/`):

```bash
ASSET_STORAGE_DRIVER=s3
AWS_REGION=ap-southeast-1
ASSET_S3_BUCKET=lingorise-assets-dev-<accountId>
ASSET_S3_PREFIX=question-assets    # optional
```

Vì bucket chặn mọi truy cập công khai, CloudFront không thể đọc nó bằng một policy public-read thông thường. Thay vào đó bạn cấp quyền cho CloudFront qua **Origin Access Control (OAC)** — bản thay thế hiện đại cho Origin Access Identity (OAI) cũ. OAC cho phép một distribution cụ thể đọc bucket riêng tư trong khi mọi bên khác vẫn bị khóa.

{{% notice info %}}
Nếu frontend của bạn lấy asset từ một origin khác với API, hãy nhớ bucket (hoặc CloudFront) cần cấu hình **CORS** cho phép domain Amplify của bạn. Trong `dev`, asset thường được tham chiếu từ cùng origin của ứng dụng, nên không cần thêm CORS — chỉ thêm khi trình duyệt báo lỗi cross-origin.
{{% /notice %}}

#### Tạo CloudFront distribution

Mở **AWS CloudFront Console** và chọn **Create distribution**.

![Tạo CloudFront distribution trong console](/images/Workshop-LingoRise/7-storage-cloudfront/cloudfront-create-distribution.png)

1. **Origin domain** — chọn bucket asset riêng tư `lingorise-assets-dev-<accountId>`.
2. **Origin access** — chọn **Origin access control settings (recommended)**, rồi **Create control setting** để tạo OAC mới. Giữ mặc định và ký request bằng SigV4.
3. Sau khi distribution được tạo, CloudFront hiển thị một **câu lệnh bucket policy** để sao chép — dán nó vào policy của bucket để CloudFront (và chỉ distribution này) có thể `s3:GetObject`. Vẫn giữ nguyên chặn truy cập công khai; OAC không yêu cầu public read.
4. **Viewer protocol policy** — **Redirect HTTP to HTTPS** (chỉ HTTPS).
5. **Cache behaviour** — tôn trọng các header `Cache-Control` mà backend gắn lên mỗi object, để asset được cache tại edge đúng vòng đời dự kiến.

![Cấu hình Origin Access Control tới bucket riêng tư](/images/Workshop-LingoRise/7-storage-cloudfront/cloudfront-oac.png)

Khi distribution triển khai xong (mất vài phút), ghi lại tên domain của nó — dạng như `d123abc.cloudfront.net`. Đó là host mà các URL có chữ ký của bạn sẽ trỏ tới.

{{% notice tip %}}
Vì CloudFront là dịch vụ toàn cầu, distribution được quản lý ở phạm vi console toàn cầu (N. Virginia) mặc dù bucket origin của bạn nằm ở `ap-southeast-1`. Region của bucket không đổi — chỉ control plane của CloudFront là toàn cầu.
{{% /notice %}}

#### URL có chữ ký / asset riêng tư

Với CloudFront đặt trước một bucket riêng tư, bạn muốn chỉ học viên đã được cấp quyền và đăng nhập mới lấy được asset. **URL có chữ ký** của CloudFront giải quyết điều này: backend ký mỗi URL asset bằng khóa riêng, và CloudFront từ chối bất kỳ request nào có chữ ký thiếu, bị chỉnh sửa, hoặc đã hết hạn.

Trước tiên, tạo một cặp khóa CloudFront:

1. AWS Console → **CloudFront → Public keys → Create public key**, tải lên một PEM public key vừa tạo.
2. Giữ PEM **private** tương ứng trên host backend, hoặc trong **AWS Secrets Manager** / SSM — không bao giờ commit nó.
3. Thêm public key vào một **Key group**, rồi đặt key group đó làm **trusted key groups** cho behaviour của distribution.

Sau đó trỏ backend tới CloudFront bằng cách đặt các biến này (trong SSM tại `/lingorise/dev/`):

```bash
ASSET_URL_MODE=cloudfront_signed
CLOUDFRONT_DOMAIN=d123abc.cloudfront.net
CLOUDFRONT_KEY_PAIR_ID=K2EXAMPLE
# Pick ONE of:
CLOUDFRONT_PRIVATE_KEY_PATH=/etc/lingorise/cloudfront.pem
CLOUDFRONT_PRIVATE_KEY_BASE64=<paste base64 of the PEM>
CLOUDFRONT_SIGNED_URL_TTL_SECONDS=3600
```

{{% notice tip %}}
Trên Lambda, hệ thống tệp là chỉ-đọc, nên hãy ưu tiên **`CLOUDFRONT_PRIVATE_KEY_BASE64`** thay vì đường dẫn tệp. Để xoay khóa, đổi biến môi trường và khóa trong CloudFront — backend đọc khóa tại thời điểm gọi, nên không cần deploy lại. TTL của URL có chữ ký được kẹp trong khoảng `[60, 604800]` giây.
{{% /notice %}}

Chuyển đổi chế độ luôn an toàn. Giá trị `storage_url` và `storage_path` lưu trong bảng `question_assets` không phụ thuộc chế độ, nên thay đổi `ASSET_URL_MODE` có hiệu lực ngay ở phản hồi kế tiếp — không cần migration cơ sở dữ liệu, và các phiên thi đang diễn ra vẫn hoạt động vì URL được resolve lúc đọc, chứ không phải lúc bắt đầu phiên.

#### Kiểm tra

Xác nhận backend thực sự đang ký qua CloudFront bằng cách gọi endpoint health:

```bash
curl -s https://<your-api-id>.execute-api.ap-southeast-1.amazonaws.com/dev/admin/system/health \
  -H "Authorization: Bearer <paste-admin-token>" | jq .assetUrlMode
# → "cloudfront_signed"
```

Bảng `/admin/system-health` cũng liệt kê mọi vấn đề cấu hình (thiếu cặp khóa, base64 không đọc được, v.v.) mà không bao giờ để lộ nội dung khóa.

Cuối cùng, lấy một asset thật qua domain CloudFront và xác nhận nó trả về `200`:

```bash
curl -sI "https://d123abc.cloudfront.net/question-assets/<some-asset>?Expires=...&Signature=...&Key-Pair-Id=K2EXAMPLE"
# → HTTP/2 200
```

![Một asset được phục vụ qua domain CDN CloudFront](/images/Workshop-LingoRise/7-storage-cloudfront/asset-via-cdn.png)

{{% notice note %}}
Nếu asset trả về **403** trên trình duyệt dù đang ở chế độ `cloudfront_signed`, gần như chắc chắn cặp khóa chưa được gắn vào trusted key groups của distribution. Nếu backend log `cloudfront signed URL fallback`, khóa riêng đã tải không thành công — kiểm tra bảng issues của `/admin/system-health`. Và nếu học viên không thấy các tệp mới tải lên, xác nhận bucket policy cho phép `PutObject` của backend và domain CloudFront khớp với `ASSET_PUBLIC_BASE_URL`.
{{% /notice %}}

Đã xong phần lưu trữ và CDN, hãy sang Chương 8 để chạy kiểm thử smoke test đầu-cuối cho toàn bộ nền tảng.
