# Nota
- Este projeto foi feito para nos ajudar a criar classes a partir de itens do banco de dados.

## Como usar
- No script temos a parte de configurações onde podemos dizer qual o nome da tabela que será usada para criar as classes.

- Depois informamos se deverá gerar os getters e setters (Se marcar como `N` ele usará o lombok do java para gerar os getters e setters).

- Depois informamos os tipos que serão convertidos para a linguagem exemplo (VARCHAR2 para STRING)

### PRECISA RODAR NO BANCO ORACLE


```sql
declare 

v_result VARCHAR2(32767);
v_table VARCHAR2(4000);
v_gerar_getter_setter VARCHAR2(1);
v_number_type VARCHAR2(20);
v_text_type VARCHAR2(20);
v_date_type VARCHAR2(20);

BEGIN 

--Configurações
v_table := 'CARGO_CLIENTE_USUARIO';
v_gerar_getter_setter := 'S';
v_number_type := 'int';
v_text_type := 'String';
v_date_type := 'LocalDateTime';


IF v_gerar_getter_setter != 'S' THEN
   v_result := 'import lombok.Data;' || chr(10) || chr(10);
   v_result := v_result || '@Data' || chr(10);
END IF;

v_result := v_result || 'public class ' || INITCAP(lower(v_table)) || ' {' || chr(10);

FOR i IN (SELECT table_name,
                 column_name,
                 data_type
            FROM USER_TAB_COLUMNS
           WHERE UPPER(table_name) = UPPER(v_table)) LOOP 
           
   v_result := v_result || '   private ' || 
   case when lower(I.data_type) = 'number' then v_number_type
        when lower(I.data_type) = 'varchar2' then v_text_type
        when lower(I.data_type) = 'date' then v_date_type
         end || ' ' || lower(i.column_name) || '; ' || chr(10);
END LOOP;

-- Adicionar os gets e Sets
IF v_gerar_getter_setter = 'S' THEN 

   FOR i IN (SELECT table_name,
                    column_name,
                    data_type
               FROM USER_TAB_COLUMNS
              WHERE UPPER(table_name) = UPPER(v_table)) LOOP 

      v_result := v_result || chr(10) || '   public ' || 
      case when lower(I.data_type) = 'number' then v_number_type
           when lower(I.data_type) = 'varchar2' then v_text_type
           when lower(I.data_type) = 'date' then v_date_type
            end || ' get' || INITCAP(lower(i.column_name)) || '() {' || chr(10);
      v_result := v_result || '      return ' || lower(i.column_name) || ';' || chr(10) || '   }' || chr(10);
      v_result := v_result || '   public void set' || INITCAP(lower(i.column_name)) || '(' || 
      case when lower(I.data_type) = 'number' then v_number_type
           when lower(I.data_type) = 'varchar2' then v_text_type
           when lower(I.data_type) = 'date' then v_date_type
            end || ' ' || lower(i.column_name) || ') {' || chr(10);
      v_result := v_result || '      this.' || lower(i.column_name) || ' = ' || lower(i.column_name) || ';' || chr(10);
      v_result := v_result || '   }' || chr(10);
   END LOOP;
END IF;

v_result := v_result || '}';
DBMS_OUTPUT.PUT_LINE(v_result);
END;
```