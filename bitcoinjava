class BitcoinTracker {
    constructor() {
        this.currency = 'usd';
        this.days = 7;
        this.chart = null;
        this.init();
    }

    init() {
        this.attachEventListeners();
        this.loadData();
    }

    attachEventListeners() {
        document.querySelectorAll('.currency-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                document.querySelectorAll('.currency-btn').forEach(b => b.classList.remove('active'));
                e.target.classList.add('active');
                this.currency = e.target.dataset.currency;
                this.loadData();
            });
        });

        document.querySelectorAll('.time-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                document.querySelectorAll('.time-btn').forEach(b => b.classList.remove('active'));
                e.target.classList.add('active');
                this.days = parseInt(e.target.dataset.days);
                this.loadData();
            });
        });

        document.getElementById('refresh-btn').addEventListener('click', () => {
            this.loadData();
        });
    }

    async loadData() {
        try {
            this.showLoading(true);
            this.hideError();

            // Fetch current price data
            const currentData = await this.fetchCurrentPrice();
            this.updateStats(currentData);

            // Fetch historical data
            const historicalData = await this.fetchHistoricalData();
            this.createChart(historicalData);

            this.updateLastUpdated();
        } catch (error) {
            this.showError(error.message);
            console.error('Error loading data:', error);
        } finally {
            this.showLoading(false);
        }
    }

    async fetchCurrentPrice() {
        const response = await fetch(
            `https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=${this.currency}&include_market_cap=true&include_24hr_vol=true&include_24hr_change=true&include_high_low_24h=true`
        );

        if (!response.ok) throw new Error('Failed to fetch current price');
        return response.json();
    }

    async fetchHistoricalData() {
        const response = await fetch(
            `https://api.coingecko.com/api/v3/coins/bitcoin/market_chart?vs_currency=${this.currency}&days=${this.days}&interval=daily`
        );

        if (!response.ok) throw new Error('Failed to fetch historical data');
        return response.json();
    }

    updateStats(data) {
        const bitcoin = data.bitcoin;
        const currency = this.currency.toUpperCase();
        const symbols = { usd: '$', eur: '€', gbp: '£', jpy: '¥' };
        const symbol = symbols[this.currency] || '$';

        const currentPrice = bitcoin[this.currency];
        const change24h = bitcoin[`${this.currency}_24h_change`];
        const high24h = bitcoin[`${this.currency}_high_24h`];
        const low24h = bitcoin[`${this.currency}_low_24h`];

        document.getElementById('current-price').textContent = 
            `${symbol}${currentPrice.toLocaleString('en-US', { maximumFractionDigits: 2 })}`;

        const changeColor = change24h >= 0 ? '#4caf50' : '#f44336';
        const changeSign = change24h >= 0 ? '+' : '';
        document.getElementById('change-24h').textContent = `${changeSign}${change24h.toFixed(2)}%`;
        document.getElementById('change-24h').parentElement.style.background = 
            `linear-gradient(135deg, ${changeColor} 0%, ${changeColor}dd 100%)`;

        document.getElementById('high-24h').textContent = 
            `${symbol}${high24h.toLocaleString('en-US', { maximumFractionDigits: 2 })}`;

        document.getElementById('low-24h').textContent = 
            `${symbol}${low24h.toLocaleString('en-US', { maximumFractionDigits: 2 })}`;
    }

    createChart(data) {
        const ctx = document.getElementById('priceChart').getContext('2d');
        const prices = data.prices.map(p => p[1]);
        const dates = data.prices.map(p => new Date(p[0]).toLocaleDateString());

        const minPrice = Math.min(...prices);
        const maxPrice = Math.max(...prices);
        const avgPrice = prices.reduce((a, b) => a + b) / prices.length;

        if (this.chart) {
            this.chart.data.labels = dates;
            this.chart.data.datasets[0].data = prices;
            this.chart.update();
        } else {
            this.chart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: dates,
                    datasets: [{
                        label: `Bitcoin Price (${this.currency.toUpperCase()})`,
                        data: prices,
                        borderColor: '#667eea',
                        backgroundColor: 'rgba(102, 126, 234, 0.1)',
                        borderWidth: 2,
                        fill: true,
                        tension: 0.4,
                        pointRadius: 0,
                        pointHoverRadius: 6,
                        pointBackgroundColor: '#667eea',
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: {
                            display: true,
                            labels: {
                                font: { size: 12 },
                                padding: 15,
                            }
                        },
                        tooltip: {
                            backgroundColor: 'rgba(0,0,0,0.8)',
                            titleFont: { size: 14 },
                            bodyFont: { size: 12 },
                            padding: 12,
                            displayColors: false,
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: false,
                            ticks: {
                                callback: function(value) {
                                    return '$' + value.toLocaleString('en-US', { maximumFractionDigits: 0 });
                                }
                            }
                        }
                    }
                }
            });
        }

        document.getElementById('priceChart').style.display = 'block';
    }

    showLoading(show) {
        const loading = document.getElementById('loading');
        if (show) {
            loading.style.display = 'block';
            document.getElementById('priceChart').style.display = 'none';
        } else {
            loading.style.display = 'none';
        }
    }

    showError(message) {
        const errorEl = document.getElementById('error');
        errorEl.textContent = `Error: ${message}`;
        errorEl.style.display = 'block';
    }

    hideError() {
        document.getElementById('error').style.display = 'none';
    }

    updateLastUpdated() {
        const now = new Date();
        document.getElementById('last-updated').textContent = 
            `Last updated: ${now.toLocaleTimeString()}`;
    }
}

// Initialize the tracker when DOM is ready
document.addEventListener('DOMContentLoaded', () => {
    new BitcoinTracker();
});
