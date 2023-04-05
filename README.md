import requests
from datetime import datetime, timedelta

def search_price():
    # Solicita ao usuário o ID do produto para pesquisar
    product_id = input("Digite o ID do produto que deseja pesquisar no Mercado Livre: ")

    # Faz a requisição à API do Mercado Livre
    try:
        response = requests.get(f"https://api.mercadolibre.com/items/{product_id}")
        response.raise_for_status()
    except requests.exceptions.HTTPError as errh:
        print("Erro HTTP:", errh)
        return
    except requests.exceptions.ConnectionError as errc:
        print("Erro de conexão:", errc)
        return
    except requests.exceptions.Timeout as errt:
        print("Tempo esgotado:", errt)
        return
    except requests.exceptions.RequestException as err:
        print("Erro na requisição:", err)
        return

    # Converte a resposta JSON em um dicionário Python
    data = response.json()

    # Exibe o preço atual do produto e a variação em relação ao dia anterior
    current_price = data["price"]
    current_date = datetime.today().strftime('%Y-%m-%d')
    yesterday = datetime.today() - timedelta(days=1)
    yesterday_date = yesterday.strftime('%Y-%m-%d')

    try:
        response = requests.get(f"https://api.mercadolibre.com/items/{product_id}/price_history?date_from={yesterday_date}&date_to={current_date}")
        response.raise_for_status()
        prices = response.json()
        if prices:
            yesterday_price = prices[-1]["unit_price"]
            price_variation = current_price - yesterday_price
            print(f"Preço atual: R${current_price:.2f}")
            if price_variation > 0:
                print(f"Variação em relação ao dia anterior: +R${price_variation:.2f}")
            elif price_variation < 0:
                print(f"Variação em relação ao dia anterior: -R${abs(price_variation):.2f}")
            else:
                print("O preço não variou em relação ao dia anterior.")
        else:
            print("Não foi possível obter o histórico de preços.")
    except requests.exceptions.RequestException as err:
        print("Erro na requisição:", err)
        return

if __name__ == "__main__":
    search_price()
