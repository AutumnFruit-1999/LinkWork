# ============================================================
# linkwork-web  —  Vue 3 / Vite 构建 + Nginx 生产服务
# Build context: LinkWork/ 根目录
# ============================================================

FROM node:22-alpine AS builder
WORKDIR /app

COPY linkwork-web/package.json linkwork-web/package-lock.json* ./
RUN if [ -f package-lock.json ]; then npm ci --prefer-offline; else npm install; fi

COPY linkwork-web/ .
RUN npm run build

# ------ 生产运行 ------
FROM nginx:1.27-alpine

COPY --from=builder /app/dist /usr/share/nginx/html
COPY deploy/docker/web/nginx.conf /etc/nginx/conf.d/default.conf

RUN rm -f /etc/nginx/conf.d/default.conf.bak

EXPOSE 3003
CMD ["nginx", "-g", "daemon off;"]
