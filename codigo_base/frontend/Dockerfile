# =============================================================================
# Taller Evaluativo 2 - Fase 2: Empaquetamiento Inmutable Multi-Stage
# Frontend: studytrack-frontend
# =============================================================================

# ---------- Stage 1: Builder ----------
# TODO

FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build


# ---------- Stage 2: Runtime ----------
# TODO
FROM nginx:alpine AS runtime

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]