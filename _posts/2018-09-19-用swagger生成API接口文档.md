#### 导读

swagger是一个API框架，号称世界上最流行的API工具。它提供了API管理的全套解决方案，比如API在线编辑器，API UI展示界面，代码生成器等诸多功能。

如果想引入swagger进行API管理。目前 springfox 是一个很好的选择，它内部会自动解析Spring容器中Controller暴露出的接口，并且也提供了一个界面用于展示或调用这些API。

本文主要内容是：

1. 利用 SpringFox 库生成实时文档；
2. 利用 Swagger2Markup Maven插件生成 asciidoc 文档；
3. 利用 asciidoctor Maven插件生成 html 或 pdf 文件；

#### 开发环境
1. SpringBoot (2.0.4.RELEASE)
2. SpringFox (2.5.0)
3. Swagger2Markup (1.2.0)
4. Maven 3.5.4

#### 利用 SpringFox 库生成实时文档
添加依赖
```
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>${springfox.version}</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>${springfox.version}</version>
</dependency>
<dependency>
    <groupId>io.swagger</groupId>
    <artifactId>swagger-annotations</artifactId>
    <version>1.5.6</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-staticdocs</artifactId>
    <version>2.6.1</version>
</dependency>
<dependency>
    <groupId>io.github.swagger2markup</groupId>
    <artifactId>swagger2markup</artifactId>
    <version>1.3.1</version>
</dependency>

```
        
```
@EnableSwagger2
@Configuration
public class SwaggerConfig {
    //创建Docket Bean
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                // 排除 error 相关的 url
                .paths(Predicates.and(ant("/**"), Predicates.not(ant("/error"))))
                .build()
                .ignoredParameterTypes(ApiIgnore.class)
                .enableUrlTemplating(true);
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("API for Scheduler")
                .description("Scheduler API Description")
                .contact(new Contact("shenjia", "http://www.caimi-inc.com", "shenjia@wacai.com"))
                .license("Apache 2.0")
                .licenseUrl("http://www.apache.org/licenses/LICENSE-2.0.html")
                .version("2.0.0")
                .build();
    }
}
```

访问：http://localhost:8080/swagger-ui.html 即可看到所有API信息。

#### 利用swagger2Markup 插件来生成 Asciidoc 文档

```
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>

    <swagger2markup.version>1.2.0</swagger2markup.version>
    <springfox.version>2.5.0</springfox.version>

    <asciidoctor.input.directory>${project.basedir}/src/docs/asciidoc</asciidoctor.input.directory>

    <swagger.output.dir>${project.build.directory}/swagger</swagger.output.dir>
    <swagger.snippetOutput.dir>${project.build.directory}/asciidoc/snippets</swagger.snippetOutput.dir>
    <generated.asciidoc.directory>${project.build.directory}/asciidoc/generated</generated.asciidoc.directory>
    <asciidoctor.html.output.directory>${project.build.directory}/asciidoc/html</asciidoctor.html.output.directory>
    <asciidoctor.pdf.output.directory>${project.build.directory}/asciidoc/pdf</asciidoctor.pdf.output.directory>
    <swagger.input>${swagger.output.dir}/swagger.json</swagger.input>
</properties>

<!-- First, use the swagger2markup plugin to generate asciidoc -->
<plugin>
    <groupId>io.github.swagger2markup</groupId>
    <artifactId>swagger2markup-maven-plugin</artifactId>
    <version>${swagger2markup.version}</version>
    <dependencies>
        <dependency>
            <groupId>ca.szc.thirdparty.nl.jworks.markdown_to_asciidoc</groupId>
            <artifactId>markdown_to_asciidoc</artifactId>
            <!-- Keep in sync with markup-document-builder's dependency -->
            <version>1.0</version>
        </dependency>
        <dependency>
            <groupId>io.github.swagger2markup</groupId>
            <artifactId>swagger2markup</artifactId>
            <!-- Keep in sync with swagger2markup-maven-plugin's dependency -->
            <version>${swagger2markup.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>nl.jworks.markdown_to_asciidoc</groupId>
                    <artifactId>markdown_to_asciidoc</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>io.github.swagger2markup</groupId>
            <artifactId>swagger2markup-import-files-ext</artifactId>
            <version>${swagger2markup.version}</version>
        </dependency>
        <dependency>
            <groupId>io.github.swagger2markup</groupId>
            <artifactId>swagger2markup-spring-restdocs-ext</artifactId>
            <version>${swagger2markup.version}</version>
        </dependency>
    </dependencies>
    <configuration>
        <swaggerInput>${swagger.input}</swaggerInput>
        <outputDir>${generated.asciidoc.directory}</outputDir>
        <config>
            <swagger2markup.markupLanguage>ASCIIDOC</swagger2markup.markupLanguage>
            <swagger2markup.pathsGroupedBy>TAGS</swagger2markup.pathsGroupedBy>

            <swagger2markup.extensions.dynamicOverview.contentPath>
                ${project.basedir}/src/docs/asciidoc/extensions/overview
            </swagger2markup.extensions.dynamicOverview.contentPath>
            <swagger2markup.extensions.dynamicDefinitions.contentPath>
                ${project.basedir}/src/docs/asciidoc/extensions/definitions
            </swagger2markup.extensions.dynamicDefinitions.contentPath>
            <swagger2markup.extensions.dynamicPaths.contentPath>
                ${project.basedir}/src/docs/asciidoc/extensions/paths
            </swagger2markup.extensions.dynamicPaths.contentPath>
            <swagger2markup.extensions.dynamicSecurity.contentPath>
                ${project.basedir}src/docs/asciidoc/extensions/security/
            </swagger2markup.extensions.dynamicSecurity.contentPath>

            <swagger2markup.extensions.springRestDocs.snippetBaseUri>${swagger.snippetOutput.dir}
            </swagger2markup.extensions.springRestDocs.snippetBaseUri>
            <swagger2markup.extensions.springRestDocs.defaultSnippets>true
            </swagger2markup.extensions.springRestDocs.defaultSnippets>
        </config>
    </configuration>
    <executions>
        <execution>
            <phase>test</phase>
            <goals>
                <goal>convertSwagger2markup</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
#### 利用 asciidoctor Maven插件生成 html 或 pdf 文件
```
 <!-- Run the generated asciidoc through Asciidoctor to generate
                 other documentation types, such as PDFs or HTML5 -->
    <plugin>
        <groupId>org.asciidoctor</groupId>
        <artifactId>asciidoctor-maven-plugin</artifactId>
        <!-- Include Asciidoctor PDF for pdf generation -->
        <dependencies>
            <dependency>
                <groupId>org.asciidoctor</groupId>
                <artifactId>asciidoctorj-pdf</artifactId>
                <version>1.5.0-alpha.10.1</version>
            </dependency>
            <dependency>
                <groupId>org.jruby</groupId>
                <artifactId>jruby-complete</artifactId>
                <version>1.7.21</version>
            </dependency>
        </dependencies>
        <!-- Configure generic document generation settings -->
        <configuration>
            <sourceDirectory>${asciidoctor.input.directory}</sourceDirectory>
            <sourceDocumentName>index.adoc</sourceDocumentName>
            <attributes>
                <doctype>book</doctype>
                <toc>left</toc>
                <toclevels>3</toclevels>
                <numbered></numbered>
                <hardbreaks></hardbreaks>
                <sectlinks></sectlinks>
                <sectanchors></sectanchors>
                <generated>${generated.asciidoc.directory}</generated>
            </attributes>
        </configuration>
        <!-- Since each execution can only handle one backend, run
             separate executions for each desired output type -->
        <executions>
            <execution>
                <id>output-html</id>
                <phase>test</phase>
                <goals>
                    <goal>process-asciidoc</goal>
                </goals>
                <configuration>
                    <backend>html5</backend>
                    <outputDirectory>${asciidoctor.html.output.directory}</outputDirectory>
                </configuration>
            </execution>
    
            <execution>
                <id>output-pdf</id>
                <phase>test</phase>
                <goals>
                    <goal>process-asciidoc</goal>
                </goals>
                <configuration>
                    <backend>pdf</backend>
                    <outputDirectory>${asciidoctor.pdf.output.directory}</outputDirectory>
                </configuration>
            </execution>
    
        </executions>
    </plugin>
```
#### 踩过的坑
swagger2markup-maven-plugin 这个插件依赖的包在maven仓库没有对应依赖，导致无法编译。解决办法：
1. 替换原有依赖 markdown_to_asciidoc
```
<dependency>
    <groupId>ca.szc.thirdparty.nl.jworks.markdown_to_asciidoc</groupId>
    <artifactId>markdown_to_asciidoc</artifactId>
    <!-- Keep in sync with markup-document-builder's dependency -->
    <version>1.0</version>
</dependency>
```   
2. 本地安装paleo-core
```
mvn install:install-file -Dfile=～/paleo-core-0.10.1.jar -DgroupId=ch.netzwerg -DartifactId=paleo-core -Dversion=0.10.1 -Dpackaging=jar
```
#### Reference
https://yq.aliyun.com/articles/599809?utm_content=m_1000002417&do=login&accounttraceid=975558fd-4933-4f0c-9bb2-01b7f4c6a971

https://github.com/Swagger2Markup/spring-swagger2markup-demo

https://asciidoctor.org/docs/asciidoc-syntax-quick-reference/

https://www.jianshu.com/p/ecb8daa4ecf7

https://leongfeng.github.io/2017/02/20/springboot-springfox-swagger2markup-spring-restdoc
