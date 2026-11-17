# --- Étape 1 : compiler le jar ---
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app

COPY .mvn/ .mvn/
COPY mvnw pom.xml ./
# Le cache /root/.m2 survit d'un build à l'autre : les dépendances ne sont
# retéléchargées que si elles changent, même quand pom.xml est modifié.
RUN --mount=type=cache,target=/root/.m2 ./mvnw dependency:go-offline -B

COPY src/ src/
RUN --mount=type=cache,target=/root/.m2 ./mvnw package -DskipTests -B

# --- Étape 2 : découper le jar en couches ---
# Les dépendances (qui changent rarement) et ton code (qui change souvent)
# deviennent des couches Docker séparées.
FROM eclipse-temurin:21-jre-alpine AS extract
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
RUN java -Djarmode=tools -jar app.jar extract --layers --launcher --destination extracted

# --- Étape 3 : l'image finale ---
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

# Du plus stable au plus changeant : seule la dernière couche change
# quand tu modifies ton code.
COPY --from=extract /app/extracted/dependencies/ ./
COPY --from=extract /app/extracted/spring-boot-loader/ ./
COPY --from=extract /app/extracted/snapshot-dependencies/ ./
COPY --from=extract /app/extracted/application/ ./

EXPOSE 8080
# La JVM utilise au plus 75 % de la mémoire donnée au conteneur.
ENV JAVA_TOOL_OPTIONS="-XX:MaxRAMPercentage=75"
ENTRYPOINT ["java", "org.springframework.boot.loader.launch.JarLauncher"]