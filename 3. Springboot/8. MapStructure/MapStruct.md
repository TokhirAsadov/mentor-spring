# MapStruct
MapStruct – bu compile-time code generator, ya’ni u kompilyatsiya vaqtida DTO ↔ Entity kabi obyektlarni avtomatik mapper kodiga aylantirib beradi.

```xml
<!-- MapStruct -->
<dependency>
  <groupId>org.mapstruct</groupId>
  <artifactId>mapstruct</artifactId>
  <version>1.6.3</version>
</dependency>
<dependency>
  <groupId>org.mapstruct</groupId>
  <artifactId>mapstruct-processor</artifactId>
  <version>1.6.3</version>
  <scope>provided</scope>
</dependency>
```

---

```java
package uz.tohir.test.TestApplication.entity;

import com.fasterxml.jackson.annotation.JsonIgnore;
import jakarta.persistence.*;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;
import lombok.*;
import uz.tohir.test.TestApplication.entity.enums.Language;
import uz.tohir.test.TestApplication.entity.temp.AbsLongEntity;

import java.util.List;

@Getter
@Setter
@ToString(exclude = {"paragraphs"}) // Rekursiyadan qochish
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Entity(name = "subjects")
public class Subject extends AbsLongEntity {

    @Column(unique = true, name = "name")
    @Size(min = 2, max = 100,message = "Name should have at least 2 characters")
    private String name;

    @Column(unique = true, name = "code")
    @Size(min = 2, max = 30,message = "Code should have at least 5 characters")
    private String code;

    // NDiscCode fan code bir xil buladi turli xil tillarda lekin umumiy code hisoblanadi
    // Cod	NDiscCod	CNaimen
    //39	5	Biologiya
    //40	5	Биология
    //41	5	Biology
    @Column(unique = true, name = "n_disc_code")
    @Size(min = 1, max = 30,message = "Code should have at least 1 characters")
    private String NDiscCode;

    @Column( name = "lang")
    @NotNull(message = "Language must not be null, It should be EN - for English, RU - for Russian, or UZ - for Uzbek")
    @Enumerated(EnumType.STRING)
    private Language language;

    @OneToMany(mappedBy = "subject")
    @JsonIgnore
    private List<Paragraph> paragraphs;

    public @Size(min = 2, max = 100, message = "Name should have at least 2 characters") String getNameUz() {
        return name;
    }

    public void setName(@Size(min = 2, max = 100, message = "Name should have at least 2 characters") String name) {
        this.name = name;
    }
}
```
```java
package uz.tohir.test.TestApplication.entity.temp;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.hibernate.annotations.CreationTimestamp;
import org.hibernate.annotations.UpdateTimestamp;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.sql.Timestamp;
import java.util.UUID;

@Data
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
@AllArgsConstructor
@NoArgsConstructor
public abstract class AbsLongEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", unique = true)
    private Long id;

    @CreationTimestamp
    @Column(updatable = false,name = "created_at")
    private Timestamp createdAt;

    @UpdateTimestamp
    @Column(name = "updated_at")
    private Timestamp updatedAt;

    @CreatedBy
    @Column(name = "created_by")
    private UUID createdBy;

    @LastModifiedBy
    @Column(name = "updated_by")
    private UUID updatedBy;

    @Column(name = "active")
    private Boolean active=true;
    @Column(name = "deleted")
    private Boolean deleted=false;

    @PreUpdate
    protected void setUpdatedAt() {
        this.updatedAt = new Timestamp(System.currentTimeMillis());
    }

    public AbsLongEntity(Long id) {
        this.id = id;
    }

}
```

```java
package uz.tohir.test.TestApplication.entity;

import com.fasterxml.jackson.annotation.JsonIgnore;
import jakarta.persistence.*;
import jakarta.validation.constraints.Size;
import lombok.*;
import uz.tohir.test.TestApplication.entity.temp.AbsLongEntity;

import java.util.List;

@Getter
@Setter
@ToString(exclude = {"subject", "sections"}) // Rekursiyadan qochish
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Entity(name = "paragraphs")
public class Paragraph extends AbsLongEntity {
    @Column(unique = true, name = "name")
    @Size(min = 1, max = 100, message = "Name should have at least 1 characters")
    private String name;

    @Column(unique = true, name = "code")
    @Size(min = 1, max = 30, message = "Code should have at least 1 characters")
    private String code;

    @ManyToOne(fetch = FetchType.LAZY) // EAGER emas, LAZY qilish kerak
    @JoinColumn(name = "subject_id")
    @JsonIgnore
    private Subject subject;

    @OneToMany(mappedBy = "paragraph", cascade = CascadeType.ALL, orphanRemoval = true)
    @JsonIgnore
    private List<Section> sections;

    public Paragraph(Long id, String name) {
        super(id);
        this.name = name;
    }

}
```
```java
package uz.tohir.test.TestApplication.entity;

import com.fasterxml.jackson.annotation.JsonIgnore;
import jakarta.persistence.*;
import jakarta.validation.constraints.Size;
import lombok.*;
import uz.tohir.test.TestApplication.entity.temp.AbsLongEntity;

@Getter
@Setter
@ToString(exclude = {"paragraph"}) // Rekursiyadan qochish
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Entity(name = "sections")
public class Section extends AbsLongEntity {
    @Column(unique = true, name = "name")
    @Size(min = 1, max = 100,message = "Name should have at least 1 characters")
    private String name;

    @Column(unique = true, name = "code")
    @Size(min = 1, max = 30,message = "Code should have at least 1 characters")
    private String code;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "paragraph_id")
    @JsonIgnore
    private Paragraph paragraph;

    public Section(Long id, String name) {
        super(id);
        this.name = name;
    }

}
```

----

```java
package uz.tohir.test.TestApplication.mapper;

import org.mapstruct.Context;
import org.mapstruct.Mapper;
import org.mapstruct.Mapping;
import org.mapstruct.Named;
import uz.tohir.test.TestApplication.entity.Paragraph;
import uz.tohir.test.TestApplication.entity.Section;
import uz.tohir.test.TestApplication.payload.section.SectionCreator;
import uz.tohir.test.TestApplication.payload.section.SectionResponse;
import uz.tohir.test.TestApplication.repository.ParagraphRepository;

import java.util.List;
import java.util.stream.Collectors;

@Mapper(componentModel = "spring")
public interface SectionMapper {

    @Mapping(target = "name", source = "creator.name")
    @Mapping(target = "code", source = "creator.code")
    @Mapping(target = "paragraph", source = "creator.paragraphId", qualifiedByName = "mapParagraph")
    Section toEntity(SectionCreator creator, @Context ParagraphRepository paragraphRepository);

    @Mapping(target = "id", source = "section.id")
    @Mapping(target = "name", source = "section.name")
    @Mapping(target = "code", source = "section.code")
    SectionResponse toDto(Section section);

    @Named("mapParagraph")
    default Paragraph mapParagraph(Long paragraphId, @Context ParagraphRepository paragraphRepository) {
        return paragraphRepository.findById(paragraphId).orElseThrow(() ->
                new RuntimeException("Paragraph not found with id: " + paragraphId));
    }

    @Named(value = "toDtoList")
    default List<SectionResponse> toDtoList(List<Section> all){
        return all.stream().map(this::toDto).collect(Collectors.toList());
    };
}
```

```java
package uz.tohir.test.TestApplication.mapper;

import org.mapstruct.Mapper;
import org.mapstruct.Mapping;
import org.mapstruct.Named;
import uz.tohir.test.TestApplication.entity.Paragraph;
import uz.tohir.test.TestApplication.entity.Section;
import uz.tohir.test.TestApplication.entity.Subject;
import uz.tohir.test.TestApplication.payload.paragraph.ParagraphCreator;
import uz.tohir.test.TestApplication.payload.paragraph.ParagraphResponse;
import uz.tohir.test.TestApplication.payload.section.SectionResponse;

import java.util.List;
import java.util.stream.Collectors;

@Mapper(componentModel = "spring")
public interface ParagraphMapper {

    @Mapping(target = "name", source = "creator.name")
    @Mapping(target = "code", source = "creator.code")
    Paragraph toEntity(ParagraphCreator creator, Subject subject);

    @Mapping(target = "id", source = "paragraph.id")
    @Mapping(target = "name", source = "paragraph.name")
    @Mapping(target = "code", source = "paragraph.code")
    @Mapping(target = "sections", source = "paragraph.sections", qualifiedByName = "toSectionList")
    ParagraphResponse toDto(Paragraph paragraph);

    @Named("toSectionList")
    default List<SectionResponse> toSectionList(List<Section> sections) {
        return sections.stream()
                .map(s -> new SectionResponse(s.getId(), s.getName(), s.getCode()))
                .collect(Collectors.toList());
    }

    @Named(value = "toDtoList")
    default List<ParagraphResponse> toDtoList(List<Paragraph> paragraphs){
        return paragraphs.stream().map(this::toDto).collect(Collectors.toList());
    };
}

```
```java
package uz.tohir.test.TestApplication.mapper;

import org.mapstruct.*;
import uz.tohir.test.TestApplication.entity.Paragraph;
import uz.tohir.test.TestApplication.entity.Section;
import uz.tohir.test.TestApplication.entity.Subject;
import uz.tohir.test.TestApplication.payload.paragraph.ParagraphResponse;
import uz.tohir.test.TestApplication.payload.section.SectionResponse;
import uz.tohir.test.TestApplication.payload.subjectDto.SubjectCreator;
import uz.tohir.test.TestApplication.payload.subjectDto.SubjectResponse;
import uz.tohir.test.TestApplication.payload.subjectDto.UpdateSubjectDto;

import java.util.List;
import java.util.stream.Collectors;

@Mapper(componentModel = "spring")
public interface SubjectMapper {

    @Mapping(target = "language", source = "creator.lang")
    Subject toEntity(SubjectCreator creator);

    @Mapping(target = "id", ignore = true)
    @Mapping(target = "language", source = "creator.lang")
    void toEntity(@MappingTarget Subject subject, SubjectCreator creator);

    @Mapping(target = "id", ignore = true)
    @Mapping(target = "language", source = "creator.lang")
    void toEntity(@MappingTarget Subject subject, UpdateSubjectDto creator);

    @Mapping(target = "lang", source = "subject.language")
    @Mapping(target = "paragraphs", source = "subject.paragraphs", qualifiedByName = "toParagraphList")
    SubjectResponse toDto(Subject subject);

    @Named("toParagraphList")
    default List<ParagraphResponse> toParagraphList(List<Paragraph> paragraphs) {
        return paragraphs.stream()
                .map(p -> new ParagraphResponse(
                        p.getId(),
                        p.getName(),
                        p.getCode(),
                        toSectionList(p.getSections()) // Map sections inside each paragraph
                ))
                .collect(Collectors.toList());
    }

    @Named("toSectionList")
    default List<SectionResponse> toSectionList(List<Section> sections) {
        return sections.stream()
                .map(s -> new SectionResponse(s.getId(), s.getName(), s.getCode()))
                .collect(Collectors.toList());
    }

    @Named(value = "toDtoList")
    default List<SubjectResponse> toDtoList(List<Subject> subjects) {
        return subjects.stream().map(this::toDto).collect(Collectors.toList());
    }

}
```

---

```java
package uz.tohir.test.TestApplication.payload.subjectDto;

import jakarta.validation.constraints.Size;
import uz.tohir.test.TestApplication.entity.enums.Language;
import uz.tohir.test.TestApplication.validation.enums.ValidLanguage;
import uz.tohir.test.TestApplication.validation.subject.ValidSubjectCreator;

@ValidSubjectCreator
public record SubjectCreator(
        @Size(min = 2, max = 100,message = "Name should have at least 2 characters")
        String name,
        @Size(min = 2, max = 30,message = "Code should have at least 2 characters")
        String code,
        @ValidLanguage Language lang) {
}
```

```java
package uz.tohir.test.TestApplication.payload.subjectDto;

import jakarta.validation.constraints.Size;
import uz.tohir.test.TestApplication.entity.enums.Language;
import uz.tohir.test.TestApplication.validation.enums.ValidLanguage;
import uz.tohir.test.TestApplication.validation.subject.ValidSubjectId;
import uz.tohir.test.TestApplication.validation.subject.ValidUpdateSubject;

@ValidUpdateSubject
public record UpdateSubjectDto(
        @ValidSubjectId
        Long id,
        @Size(min = 2, max = 100, message = "Name should have at least 2 characters")
        String name,
        @Size(min = 2, max = 30, message = "Code should have at least 2 characters")
        String code,
        @ValidLanguage Language lang
) {}
```
```java
package uz.tohir.test.TestApplication.payload.paragraph;

import jakarta.validation.constraints.Size;
import uz.tohir.test.TestApplication.validation.paragraph.ValidParagraphCreator;
import uz.tohir.test.TestApplication.validation.subject.ValidSubjectId;

@ValidParagraphCreator
public record ParagraphCreator(
        @Size(min = 1, max = 100, message = "Name should have at least 1 characters")
        String name,
        @Size(min = 1, max = 30, message = "Code should have at least 1 characters")
        String code,
        @ValidSubjectId Long subjectId) {
}
```
```java
package uz.tohir.test.TestApplication.payload.section;

import jakarta.validation.constraints.Size;
import uz.tohir.test.TestApplication.validation.paragraph.ValidParagraphId;
import uz.tohir.test.TestApplication.validation.section.ValidSectionCreator;

@ValidSectionCreator
public record SectionCreator(
        @Size(min = 1, max = 100,message = "Name should have at least 1 character")
        String name,
        @Size(min = 1, max = 30,message = "Code should have at least 1 character")
        String code,
        @ValidParagraphId Long paragraphId) {
}
```

