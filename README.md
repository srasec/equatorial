import undetected_chromedriver as uc
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from webdriver_manager.chrome import ChromeDriverManager
import time

try:
    # Usa o WebDriverManager para baixar e instalar a versão correta do ChromeDriver
    driver = uc.Chrome(driver_executable_path=ChromeDriverManager().install())

    # Acessar a página
    driver.get("https://ma.equatorialenergia.com.br/sua-conta/emitir-segunda-via/")
    time.sleep(5)  # Esperar a página carregar completamente

    print("Página carregada. Tentando localizar o botão...")

    # Localizar e clicar no botão CONTINUAR NO SITE com espera de até 30 segundos
    continuar_xpath = '/html/body/div[8]/div[2]/div/div/div/div/div/div/div/div[4]/button'

    try:
        continuar_button = WebDriverWait(driver, 30).until(EC.element_to_be_clickable((By.XPATH, continuar_xpath)))
        driver.execute_script("arguments[0].scrollIntoView(true);", continuar_button)
        print("Botão encontrado. Clicando...")
        continuar_button.click()
    except Exception as e:
        print(f"Erro ao tentar clicar no botão: {e}")

    # Marcar o checkbox "Li e entendi o Aviso de Privacidade"
    try:
        checkbox = WebDriverWait(driver, 20).until(EC.element_to_be_clickable((By.ID, "aviso_aceite")))
        driver.execute_script("arguments[0].scrollIntoView(true);", checkbox)
        checkbox.click()
    except Exception as e:
        print(f"Erro ao marcar o checkbox: {e}")

    # Clicar no botão Enviar após o aceite
    enviar_xpath = '//*[@id="lgpd_accept"]'
    try:
        enviar_button = WebDriverWait(driver, 20).until(EC.element_to_be_clickable((By.XPATH, enviar_xpath)))
        driver.execute_script("arguments[0].scrollIntoView(true);", enviar_button)
        enviar_button.click()
    except Exception as e:
        print(f"Erro ao clicar no botão 'Enviar': {e}")

    # Preencher o campo do CPF
    cpf_xpath = '//*[@id="identificador-otp"]'
    try:
        cpf_field = WebDriverWait(driver, 20).until(EC.presence_of_element_located((By.XPATH, cpf_xpath)))
        driver.execute_script("arguments[0].scrollIntoView(true);", cpf_field)
        cpf_field.send_keys("855.219.143-15")
    except Exception as e:
        print(f"Erro ao preencher o CPF: {e}")

    # Confirmar o CPF
    confirmar_cpf_xpath = '//*[@id="envia-identificador-otp"]'
    try:
        confirmar_cpf_button = WebDriverWait(driver, 20).until(EC.element_to_be_clickable((By.XPATH, confirmar_cpf_xpath)))
        driver.execute_script("arguments[0].scrollIntoView(true);", confirmar_cpf_button)
        confirmar_cpf_button.click()
    except Exception as e:
        print(f"Erro ao confirmar o CPF: {e}")

    # Preencher o campo da data de nascimento 23/07/1980
    data_nascimento_xpath = '//*[@id="senha-identificador"]'
    try:
        # Adicionar um atraso para garantir que o campo carregue completamente
        time.sleep(10)

        # Verificar se o campo está disponível e interativo
        data_nascimento_field = WebDriverWait(driver, 20).until(EC.element_to_be_clickable((By.XPATH, data_nascimento_xpath)))
        driver.execute_script("arguments[0].scrollIntoView(true);", data_nascimento_field)
        data_nascimento_field.send_keys("23/07/1980")
        print("Data de nascimento preenchida com sucesso.")
    except Exception as e:
        print(f"Erro ao preencher a data de nascimento: {e}")

    # Confirmar a data de nascimento
    confirmar_data_xpath = '//*[@id="envia-identificador"]'
    try:
        confirmar_data_button = WebDriverWait(driver, 20).until(EC.element_to_be_clickable((By.XPATH, confirmar_data_xpath)))
        driver.execute_script("arguments[0].scrollIntoView(true);", confirmar_data_button)
        confirmar_data_button.click()
        print("Data de nascimento confirmada.")
    except Exception as e:
        print(f"Erro ao confirmar a data de nascimento: {e}")

    # Atualizar a página após o login
    print("Login realizado com sucesso. Atualizando a página em 5 segundos...")
    time.sleep(5)  # Aguarda 5 segundos antes de atualizar
    driver.refresh()
    print("Página atualizada com sucesso.")
    
    # Raspar faturas de um período específico
    try:
        # Esperar que a seção das faturas seja carregada
        WebDriverWait(driver, 20).until(EC.presence_of_element_located((By.XPATH, '//*[@id="list-bills-segunda-via"]/tbody')))

        # Definir o período que queremos filtrar (por exemplo, de 08/2017 a 07/2024)
        periodo_inicio = "08/2017"
        periodo_fim = "07/2024"

        # Raspar todas as linhas da tabela
        linhas_faturas = driver.find_elements(By.XPATH, '//*[@id="list-bills-segunda-via"]/tbody/tr')

        for linha in linhas_faturas:
            # Extrair a data de referência de cada linha
            referencia = linha.find_element(By.XPATH, './td[1]').text

            # Comparar a data de referência com o período desejado
            if periodo_inicio <= referencia <= periodo_fim:
                # Se a data estiver no período, pegar o link para o download
                link_fatura = linha.find_element(By.XPATH, './td[3]/a').get_attribute('href')
                print(f"Baixando fatura de {referencia}...")
                driver.get(link_fatura)
                time.sleep(2)  # Esperar o download
    except Exception as e:
        print(f"Erro ao raspar as faturas: {e}")

    # O navegador permanecerá aberto após o término da execução do script
    input("Pressione Enter para fechar o navegador e encerrar o programa...")

finally:
    # Remover o driver manualmente no final, se necessário
    pass

