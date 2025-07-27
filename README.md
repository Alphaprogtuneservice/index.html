import React, { useState, useEffect } from 'react';
import { Search, Car, Calendar, FileText, Plus, Trash2, Edit3, Save, X } from 'lucide-react';

const TuningManagementApp = () => {
  const [currentView, setCurrentView] = useState('search');
  const [selectedVehicle, setSelectedVehicle] = useState(null);
  const [clients, setClients] = useState([]);
  const [searchTerm, setSearchTerm] = useState('');
  const [editingClient, setEditingClient] = useState(null);

  // Base de données véhicules (simulée - en réalité connectée à une API)
  const vehicleDatabase = {
    'AUDI A3 2019 2.0 TDI': {
      marque: 'AUDI',
      modele: 'A3',
      annee: '2019',
      motorisation: '2.0 TDI',
      typeMoteur: 'CUNA',
      calculateur: 'Bosch EDC17CP14',
      carburant: 'Diesel',
      puissanceOrigine: '150 ch',
      coupleOrigine: '340 Nm',
      turbo: 'Oui - VTG',
      dimensionsPneus: '225/45 R17',
      transmission: 'Manuel 6 vitesses',
      norme: 'Euro 6d-TEMP',
      dpf: 'Oui',
      egr: 'Oui',
      adblue: 'Oui',
      stage1Possible: '185 ch / 400 Nm',
      stage2Possible: '200 ch / 430 Nm',
      stage3Possible: 'Sur devis - turbo upgrade'
    },
    'BMW 320D 2020 2.0D': {
      marque: 'BMW',
      modele: '320d',
      annee: '2020',
      motorisation: '2.0d',
      typeMoteur: 'B47D20',
      calculateur: 'Bosch DDE8.1',
      carburant: 'Diesel',
      puissanceOrigine: '190 ch',
      coupleOrigine: '400 Nm',
      turbo: 'Oui - Twin Turbo',
      dimensionsPneus: '225/50 R17',
      transmission: 'Automatique 8 vitesses',
      norme: 'Euro 6d-TEMP',
      dpf: 'Oui',
      egr: 'Oui',
      adblue: 'Oui',
      stage1Possible: '230 ch / 470 Nm',
      stage2Possible: '250 ch / 500 Nm',
      stage3Possible: 'Sur devis - upgrade turbo'
    },
    'MERCEDES C220 2021 2.0 CDI': {
      marque: 'MERCEDES-BENZ',
      modele: 'C220 CDI',
      annee: '2021',
      motorisation: '2.0 CDI',
      typeMoteur: 'OM654',
      calculateur: 'Bosch EDC17CP57',
      carburant: 'Diesel',
      puissanceOrigine: '194 ch',
      coupleOrigine: '400 Nm',
      turbo: 'Oui - VTG',
      dimensionsPneus: '225/55 R17',
      transmission: 'Automatique 9G-TRONIC',
      norme: 'Euro 6d',
      dpf: 'Oui',
      egr: 'Oui',
      adblue: 'Oui',
      stage1Possible: '240 ch / 480 Nm',
      stage2Possible: '260 ch / 520 Nm',
      stage3Possible: 'Sur devis - modifications hardware'
    },
    'VOLKSWAGEN GOLF 2018 2.0 TDI': {
      marque: 'VOLKSWAGEN',
      modele: 'Golf',
      annee: '2018',
      motorisation: '2.0 TDI',
      typeMoteur: 'CUNA',
      calculateur: 'Bosch EDC17CP20',
      carburant: 'Diesel',
      puissanceOrigine: '150 ch',
      coupleOrigine: '340 Nm',
      turbo: 'Oui - VTG',
      dimensionsPneus: '205/55 R16',
      transmission: 'Manuel 6 vitesses',
      norme: 'Euro 6b',
      dpf: 'Oui',
      egr: 'Oui',
      adblue: 'Non',
      stage1Possible: '190 ch / 400 Nm',
      stage2Possible: '210 ch / 440 Nm',
      stage3Possible: 'Sur devis - turbo hybrid'
    }
  };

  const noTuneOptions = [
    { name: 'EGR OFF', price: 90, description: 'Suppression vanne EGR' },
    { name: 'DPF OFF', price: 90, description: 'Suppression filtre à particules' },
    { name: 'VMAX OFF', price: 90, description: 'Suppression limiteur vitesse' },
    { name: 'LAMBDA OFF', price: 90, description: 'Suppression sondes lambda' },
    { name: 'ADBLUE OFF', price: 90, description: 'Suppression système AdBlue' },
    { name: 'HARDCUT', price: 90, description: 'Coupure dure à hauts régimes' }
  ];

  const [selectedOptions, setSelectedOptions] = useState({
    stage: '',
    noTuneOptions: []
  });

  const [clientForm, setClientForm] = useState({
    nom: '',
    prenom: '',
    email: '',
    telephone: '',
    adresse: '',
    dateRdv: '',
    heureRdv: '',
    vehicule: null,
    options: { stage: '', noTuneOptions: [] },
    total: 0,
    notes: ''
  });

  const searchVehicles = (term) => {
    if (!term) return [];
    return Object.keys(vehicleDatabase).filter(key =>
      key.toLowerCase().includes(term.toLowerCase())
    );
  };

  const calculateTotal = (stage, options) => {
    let total = 0;
    
    if (stage === 'Stage 1') total += 280;
    else if (stage === 'Stage 2') total += 300;
    else if (stage === 'Stage 3') return 'Sur devis';

    // Offres spéciales pour Stage 1
    if (stage === 'Stage 1') {
      const hasEGR = options.includes('EGR OFF');
      const hasDPF = options.includes('DPF OFF');
      const hasVMAX = options.includes('VMAX OFF');
      
      if (hasEGR && hasDPF) {
        // VMAX OFF offert
        const filteredOptions = options.filter(opt => 
          !['EGR OFF', 'DPF OFF', 'VMAX OFF'].includes(opt)
        );
        total += filteredOptions.length * 90;
      } else {
        total += options.length * 90;
      }
    } else {
      total += options.length * 90;
    }

    return total;
  };

  const handleOptionChange = (optionType, value) => {
    const newOptions = { ...selectedOptions };
    
    if (optionType === 'stage') {
      newOptions.stage = value;
    } else if (optionType === 'noTune') {
      if (newOptions.noTuneOptions.includes(value)) {
        newOptions.noTuneOptions = newOptions.noTuneOptions.filter(opt => opt !== value);
      } else {
        newOptions.noTuneOptions.push(value);
      }
    }
    
    setSelectedOptions(newOptions);
  };

  const saveClient = () => {
    const total = calculateTotal(selectedOptions.stage, selectedOptions.noTuneOptions);
    const newClient = {
      ...clientForm,
      id: Date.now(),
      vehicule: selectedVehicle,
      options: selectedOptions,
      total: total,
      dateCreation: new Date().toLocaleDateString('fr-FR')
    };
    
    if (editingClient) {
      setClients(clients.map(c => c.id === editingClient.id ? { ...newClient, id: editingClient.id } : c));
      setEditingClient(null);
    } else {
      setClients([...clients, newClient]);
    }
    
    // Reset form
    setClientForm({
      nom: '', prenom: '', email: '', telephone: '', adresse: '',
      dateRdv: '', heureRdv: '', vehicule: null, options: { stage: '', noTuneOptions: [] },
      total: 0, notes: ''
    });
    setSelectedOptions({ stage: '', noTuneOptions: [] });
    setSelectedVehicle(null);
    setCurrentView('clients');
  };

  const editClient = (client) => {
    setEditingClient(client);
    setClientForm(client);
    setSelectedVehicle(client.vehicule);
    setSelectedOptions(client.options);
    setCurrentView('devis');
  };

  const deleteClient = (clientId) => {
    setClients(clients.filter(c => c.id !== clientId));
  };

  const VehicleSearch = () => (
    <div className="p-6">
      <div className="max-w-4xl mx-auto">
        <h2 className="text-2xl font-bold mb-6 text-gray-800">Recherche Véhicule</h2>
        
        <div className="relative mb-6">
          <Search className="absolute left-3 top-3 h-5 w-5 text-gray-400" />
          <input
            type="text"
            placeholder="Rechercher un véhicule (ex: AUDI A3 2019 2.0 TDI)"
            className="w-full pl-10 pr-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
            value={searchTerm}
            onChange={(e) => setSearchTerm(e.target.value)}
          />
        </div>

        <div className="grid gap-4">
          {searchVehicles(searchTerm).map((vehicleKey) => (
            <div
              key={vehicleKey}
              className="bg-white p-4 border border-gray-200 rounded-lg hover:shadow-md cursor-pointer transition-shadow"
              onClick={() => {
                setSelectedVehicle({ key: vehicleKey, ...vehicleDatabase[vehicleKey] });
                setCurrentView('vehicle-details');
              }}
            >
              <div className="flex items-center justify-between">
                <div className="flex items-center space-x-3">
                  <Car className="h-8 w-8 text-blue-500" />
                  <div className="mt-4 pt-4 border-t border-gray-200">
                    <div className="flex items-center justify-between">
                      <div>
                        <p className="font-medium">Services: {client.options.stage}</p>
                        {client.options.noTuneOptions.length > 0 && (
                          <p className="text-sm text-gray-600">
                            Options: {client.options.noTuneOptions.join(', ')}
                          </p>
                        )}
                        {client.notes && (
                          <p className="text-sm text-gray-500 mt-1">Notes: {client.notes}</p>
                        )}
                      </div>
                      <div className="text-right">
                        <p className="text-2xl font-bold text-green-600">
                          {client.total}{typeof client.total === 'number' && '€'}
                        </p>
                      </div>
                    </div>
                  </div>
                </div>
                
                <div className="flex space-x-2 ml-4">
                  <button
                    onClick={() => editClient(client)}
                    className="p-2 text-blue-600 hover:bg-blue-50 rounded-lg"
                    title="Modifier"
                  >
                    <Edit3 className="h-4 w-4" />
                  </button>
                  <button
                    onClick={() => deleteClient(client.id)}
                    className="p-2 text-red-600 hover:bg-red-50 rounded-lg"
                    title="Supprimer"
                  >
                    <Trash2 className="h-4 w-4" />
                  </button>
                </div>
              </div>
            </div>
          ))}
          
          {clients.length === 0 && (
            <div className="text-center py-12">
              <FileText className="h-12 w-12 text-gray-400 mx-auto mb-4" />
              <p className="text-gray-500">Aucun client enregistré</p>
              <button
                onClick={() => setCurrentView('search')}
                className="mt-4 bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600"
              >
                Créer le premier devis
              </button>
            </div>
          )}
        </div>
      </div>
    </div>
  );

  return (
    <div className="min-h-screen bg-gray-100">
      {/* Navigation */}
      <nav className="bg-blue-600 text-white p-4">
        <div className="max-w-6xl mx-auto flex items-center justify-between">
          <h1 className="text-xl font-bold">Reprogrammation Moteur - Sangalhos</h1>
          <div className="flex space-x-4">
            <button
              onClick={() => setCurrentView('search')}
              className={`px-4 py-2 rounded-lg flex items-center space-x-2 ${
                currentView === 'search' || currentView === 'vehicle-details' || currentView === 'devis' 
                  ? 'bg-blue-700' : 'hover:bg-blue-700'
              }`}
            >
              <Search className="h-4 w-4" />
              <span>Recherche</span>
            </button>
            <button
              onClick={() => setCurrentView('clients')}
              className={`px-4 py-2 rounded-lg flex items-center space-x-2 ${
                currentView === 'clients' ? 'bg-blue-700' : 'hover:bg-blue-700'
              }`}
            >
              <FileText className="h-4 w-4" />
              <span>Clients ({clients.length})</span>
            </button>
          </div>
        </div>
      </nav>

      {/* Content */}
      {currentView === 'search' && <VehicleSearch />}
      {currentView === 'vehicle-details' && <VehicleDetails />}
      {currentView === 'devis' && <DevisForm />}
      {currentView === 'clients' && <ClientsList />}
    </div>
  );
};

export default TuningManagementApp;>
                    <h3 className="font-semibold text-gray-800">{vehicleKey}</h3>
                    <p className="text-sm text-gray-600">
                      {vehicleDatabase[vehicleKey].puissanceOrigine} • {vehicleDatabase[vehicleKey].carburant}
                    </p>
                  </div>
                </div>
                <button className="text-blue-500 hover:text-blue-700">
                  Voir détails →
                </button>
              </div>
            </div>
          ))}
        </div>

        {searchTerm && searchVehicles(searchTerm).length === 0 && (
          <div className="text-center py-8">
            <p className="text-gray-500">Aucun véhicule trouvé pour "{searchTerm}"</p>
          </div>
        )}

        {!searchTerm && (
          <div className="text-center py-8">
            <Car className="h-12 w-12 text-gray-400 mx-auto mb-4" />
            <p className="text-gray-500">Tapez pour rechercher un véhicule</p>
            <p className="text-sm text-gray-400 mt-2">Exemple: AUDI A3, BMW 320D, MERCEDES C220...</p>
          </div>
        )}
      </div>
    </div>
  );

  const VehicleDetails = () => (
    <div className="p-6">
      <div className="max-w-4xl mx-auto">
        <div className="flex items-center justify-between mb-6">
          <div className="flex items-center space-x-4">
            <button
              onClick={() => setCurrentView('search')}
              className="text-blue-500 hover:text-blue-700"
            >
              ← Retour
            </button>
            <h2 className="text-2xl font-bold text-gray-800">Fiche Technique Véhicule</h2>
          </div>
          <button
            onClick={() => setCurrentView('devis')}
            className="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600 flex items-center space-x-2"
          >
            <Plus className="h-4 w-4" />
            <span>Créer Devis</span>
          </button>
        </div>

        <div className="bg-white rounded-lg shadow-lg p-6">
          <div className="grid md:grid-cols-2 gap-6">
            <div>
              <h3 className="text-lg font-semibold mb-4 text-gray-800">Informations Générales</h3>
              <div className="space-y-3">
                <div className="flex justify-between">
                  <span className="text-gray-600">Marque:</span>
                  <span className="font-medium">{selectedVehicle.marque}</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-gray-600">Modèle:</span>
                  <span className="font-medium">{selectedVehicle.modele}</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-gray-600">Année:</span>
                  <span className="font-medium">{selectedVehicle.annee}</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-gray-600">Motorisation:</span>
                  <span className="font-medium">{selectedVehicle.motorisation}</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-gray-600">Type Moteur:</span>
                  <span className="font-medium">{selectedVehicle.typeMoteur}</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-gray-600">Calculateur:</span>
                  <span className="font-medium">{selectedVehicle.calculateur}</span>
                </div>
              </div>
            </div>

            <div>
              <h3 className="text-lg font-semibold mb-4 text-gray-800">Caractéristiques Techniques</h3>
              <div className="space-y-3">
                <div className="flex justify-between">
                  <span className="text-gray-600">Carburant:</span>
                  <span className="font-medium">{selectedVehicle.carburant}</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-gray-600">Puissance origine:</span>
                  <span className="font-medium">{selectedVehicle.puissanceOrigine}</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-gray-600">Couple origine:</span>
                  <span className="font-medium">{selectedVehicle.coupleOrigine}</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-gray-600">Turbo:</span>
                  <span className="font-medium">{selectedVehicle.turbo}</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-gray-600">Dimensions pneus:</span>
                  <span className="font-medium">{selectedVehicle.dimensionsPneus}</span>
                </div>
                <div className="flex justify-between">
                  <span className="text-gray-600">Transmission:</span>
                  <span className="font-medium">{selectedVehicle.transmission}</span>
                </div>
              </div>
            </div>
          </div>

          <div className="mt-6 pt-6 border-t border-gray-200">
            <h3 className="text-lg font-semibold mb-4 text-gray-800">Systèmes Antipollution</h3>
            <div className="grid md:grid-cols-4 gap-4">
              <div className="flex justify-between">
                <span className="text-gray-600">Norme:</span>
                <span className="font-medium">{selectedVehicle.norme}</span>
              </div>
              <div className="flex justify-between">
                <span className="text-gray-600">DPF:</span>
                <span className="font-medium">{selectedVehicle.dpf}</span>
              </div>
              <div className="flex justify-between">
                <span className="text-gray-600">EGR:</span>
                <span className="font-medium">{selectedVehicle.egr}</span>
              </div>
              <div className="flex justify-between">
                <span className="text-gray-600">AdBlue:</span>
                <span className="font-medium">{selectedVehicle.adblue}</span>
              </div>
            </div>
          </div>

          <div className="mt-6 pt-6 border-t border-gray-200">
            <h3 className="text-lg font-semibold mb-4 text-gray-800">Potentiel Reprogrammation</h3>
            <div className="grid md:grid-cols-3 gap-4">
              <div className="bg-green-50 p-4 rounded-lg">
                <span className="text-sm text-gray-600">Stage 1 - 280€</span>
                <p className="font-medium text-green-600">{selectedVehicle.stage1Possible}</p>
              </div>
              <div className="bg-orange-50 p-4 rounded-lg">
                <span className="text-sm text-gray-600">Stage 2 - 300€</span>
                <p className="font-medium text-orange-600">{selectedVehicle.stage2Possible}</p>
              </div>
              <div className="bg-red-50 p-4 rounded-lg">
                <span className="text-sm text-gray-600">Stage 3</span>
                <p className="font-medium text-red-600">{selectedVehicle.stage3Possible}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  );

  const DevisForm = () => (
    <div className="p-6">
      <div className="max-w-4xl mx-auto">
        <div className="flex items-center space-x-4 mb-6">
          <button
            onClick={() => setCurrentView(selectedVehicle ? 'vehicle-details' : 'search')}
            className="text-blue-500 hover:text-blue-700"
          >
            ← Retour
          </button>
          <h2 className="text-2xl font-bold text-gray-800">
            {editingClient ? 'Modifier Client' : 'Nouveau Devis Client'}
          </h2>
        </div>

        <div className="grid lg:grid-cols-2 gap-8">
          {/* Informations Client */}
          <div className="bg-white rounded-lg shadow-lg p-6">
            <h3 className="text-lg font-semibold mb-4">Informations Client</h3>
            <div className="space-y-4">
              <div className="grid grid-cols-2 gap-4">
                <input
                  type="text"
                  placeholder="Nom"
                  className="px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                  value={clientForm.nom}
                  onChange={(e) => setClientForm({...clientForm, nom: e.target.value})}
                />
                <input
                  type="text"
                  placeholder="Prénom"
                  className="px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                  value={clientForm.prenom}
                  onChange={(e) => setClientForm({...clientForm, prenom: e.target.value})}
                />
              </div>
              <input
                type="email"
                placeholder="Email"
                className="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                value={clientForm.email}
                onChange={(e) => setClientForm({...clientForm, email: e.target.value})}
              />
              <input
                type="tel"
                placeholder="Téléphone"
                className="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                value={clientForm.telephone}
                onChange={(e) => setClientForm({...clientForm, telephone: e.target.value})}
              />
              <textarea
                placeholder="Adresse"
                className="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                rows="3"
                value={clientForm.adresse}
                onChange={(e) => setClientForm({...clientForm, adresse: e.target.value})}
              />
              
              <div className="grid grid-cols-2 gap-4">
                <input
                  type="date"
                  className="px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                  value={clientForm.dateRdv}
                  onChange={(e) => setClientForm({...clientForm, dateRdv: e.target.value})}
                />
                <input
                  type="time"
                  className="px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                  value={clientForm.heureRdv}
                  onChange={(e) => setClientForm({...clientForm, heureRdv: e.target.value})}
                />
              </div>
            </div>
          </div>

          {/* Services et Options */}
          <div className="space-y-6">
            {/* Véhicule sélectionné */}
            {selectedVehicle ? (
              <div className="bg-white rounded-lg shadow-lg p-6">
                <h3 className="text-lg font-semibold mb-2">Véhicule</h3>
                <p className="text-gray-600">{selectedVehicle.key}</p>
                <p className="text-sm text-gray-500">{selectedVehicle.puissanceOrigine} • {selectedVehicle.calculateur}</p>
              </div>
            ) : (
              <div className="bg-yellow-50 border border-yellow-200 rounded-lg p-6">
                <p className="text-yellow-800">⚠️ Aucun véhicule sélectionné</p>
                <button
                  onClick={() => setCurrentView('search')}
                  className="mt-2 text-blue-500 hover:text-blue-700 underline"
                >
                  Rechercher un véhicule
                </button>
              </div>
            )}

            {/* Stages */}
            <div className="bg-white rounded-lg shadow-lg p-6">
              <h3 className="text-lg font-semibold mb-4">Stages de Reprogrammation</h3>
              <div className="space-y-3">
                {['Stage 1', 'Stage 2', 'Stage 3'].map((stage) => (
                  <label key={stage} className="flex items-center space-x-3 cursor-pointer">
                    <input
                      type="radio"
                      name="stage"
                      value={stage}
                      checked={selectedOptions.stage === stage}
                      onChange={(e) => handleOptionChange('stage', e.target.value)}
                      className="h-4 w-4 text-blue-600"
                    />
                    <span className="font-medium">{stage}</span>
                    <span className="text-gray-600">
                      - {stage === 'Stage 1' ? '280€' : stage === 'Stage 2' ? '300€' : 'Sur devis'}
                    </span>
                  </label>
                ))}
              </div>
            </div>

            {/* Options No Tune */}
            <div className="bg-white rounded-lg shadow-lg p-6">
              <h3 className="text-lg font-semibold mb-4">Options "No Tune" (90€ chacune)</h3>
              {selectedOptions.stage === 'Stage 1' && (
                <div className="mb-4 p-3 bg-green-50 border border-green-200 rounded-lg">
                  <p className="text-sm text-green-700">
                    🎁 Offre Stage 1: EGR OFF + DPF OFF = VMAX OFF OFFERT !
                  </p>
                </div>
              )}
              <div className="space-y-3">
                {noTuneOptions.map((option) => (
                  <label key={option.name} className="flex items-start space-x-3 cursor-pointer">
                    <input
                      type="checkbox"
                      checked={selectedOptions.noTuneOptions.includes(option.name)}
                      onChange={() => handleOptionChange('noTune', option.name)}
                      className="h-4 w-4 text-blue-600 mt-1"
                    />
                    <div>
                      <span className="font-medium">{option.name}</span>
                      <p className="text-sm text-gray-600">{option.description}</p>
                    </div>
                  </label>
                ))}
              </div>
            </div>

            {/* Total */}
            <div className="bg-blue-50 border border-blue-200 rounded-lg p-6">
              <div className="flex justify-between items-center">
                <span className="text-lg font-semibold">Total:</span>
                <span className="text-2xl font-bold text-blue-600">
                  {calculateTotal(selectedOptions.stage, selectedOptions.noTuneOptions)}
                  {typeof calculateTotal(selectedOptions.stage, selectedOptions.noTuneOptions) === 'number' && '€'}
                </span>
              </div>
            </div>

            {/* Notes */}
            <div className="bg-white rounded-lg shadow-lg p-6">
              <h3 className="text-lg font-semibold mb-4">Notes</h3>
              <textarea
                placeholder="Notes complémentaires..."
                className="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                rows="3"
                value={clientForm.notes}
                onChange={(e) => setClientForm({...clientForm, notes: e.target.value})}
              />
            </div>

            {/* Boutons */}
            <div className="flex space-x-4">
              <button
                onClick={saveClient}
                disabled={!clientForm.nom || !clientForm.prenom || !selectedVehicle}
                className="flex-1 bg-green-500 text-white px-4 py-3 rounded-lg hover:bg-green-600 disabled:bg-gray-400 disabled:cursor-not-allowed flex items-center justify-center space-x-2"
              >
                <Save className="h-4 w-4" />
                <span>{editingClient ? 'Modifier' : 'Enregistrer'} Client</span>
              </button>
              <button
                onClick={() => {
                  setCurrentView('search');
                  setSelectedVehicle(null);
                  setEditingClient(null);
                  setSelectedOptions({ stage: '', noTuneOptions: [] });
                }}
                className="px-4 py-3 border border-gray-300 rounded-lg hover:bg-gray-50 flex items-center space-x-2"
              >
                <X className="h-4 w-4" />
                <span>Annuler</span>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  );

  const ClientsList = () => (
    <div className="p-6">
      <div className="max-w-6xl mx-auto">
        <h2 className="text-2xl font-bold mb-6 text-gray-800">Gestion Clients</h2>
        
        <div className="grid gap-4">
          {clients.map((client) => (
            <div key={client.id} className="bg-white rounded-lg shadow-md p-6">
              <div className="flex justify-between items-start">
                <div className="flex-1">
                  <div className="grid md:grid-cols-3 gap-4">
                    <div>
                      <h3 className="font-semibold text-lg">{client.prenom} {client.nom}</h3>
                      <p className="text-gray-600">{client.email}</p>
                      <p className="text-gray-600">{client.telephone}</p>
                    </div>
                    
                    <div>
                      <p className="font-medium text-gray-800">Véhicule:</p>
                      <p className="text-gray-600">{client.vehicule?.key}</p>
                      <p className="text-sm text-gray-500">{client.vehicule?.puissanceOrigine}</p>
                    </div>
                    
                    <div>
                      <p className="font-medium text-gray-800">Rendez-vous:</p>
                      <p className="text-gray-600">
                        {client.dateRdv} à {client.heureRdv}
                      </p>
                      <p className="text-sm text-gray-500">Créé le {client.dateCreation}</p>
                    </div>
                  </div>
                  
                  <div
