Build resume as PDF with asciidoctor-pdf

asciidoctor-pdf -b pdf -a pdf-theme=resume -a pdf-themesdir=content/themes -a imagesdir=../../content/images content/resume/vincent_gourrierec.adoc


Build en resume 

 asciidoctor-pdf -b pdf -a pdf-theme=resume -a pdf-themesdir=content/themes -a imagesdir=../../../content/images content/resume/en/vincent_gourrierec.adoc -a icons=font -a icon-set=fas -o vincent_gourrierec_en.pdf